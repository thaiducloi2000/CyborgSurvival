# Modular Cyborg Character — Kiến trúc hệ thống (Unreal Engine / C++)

> Ghi chú thiết kế trước khi code — game Survival Cyborg, body part tùy chỉnh mang chỉ số & công dụng khác nhau.

## 1. Bối cảnh & 3 kỹ thuật modular character của Unreal

| Kỹ thuật | Cơ chế | Chi phí | Khi dùng |
|---|---|---|---|
| **Leader Pose Component** (Master Pose cũ) | Follower `SkeletalMeshComponent` copy trực tiếp bone transform từ leader, không chạy AnimBP riêng | Rẻ | Mặc định cho mọi part dùng chung skeleton, không cần animation riêng |
| **Copy Pose from Mesh** | Node trong AnimGraph, follower vẫn chạy AnimBP riêng, nhận pose từ mesh khác rồi blend thêm layer riêng | Trung bình–cao | Part đặc biệt cần animation phụ (blade thu/duỗi, turret tự xoay, hiệu ứng hư hỏng) |
| **Skeletal Mesh Merge** | Gộp nhiều mesh thành 1 mesh tĩnh, giảm draw call, mất khả năng swap linh hoạt | Bake 1 lần | NPC có loadout cố định, LOD xa, player khác trong multiplayer khi không cần swap real-time |

**Kết hợp cả 3 theo tầng** là hướng phù hợp, không phải chọn 1:
- Player chính: Leader Pose làm nền + Copy Pose cho part hiếm/có mechanic riêng.
- NPC / character ở xa / nhiều người chơi khác hiển thị cùng lúc: Mesh Merge, cache theo tổ hợp part để tránh bake lại liên tục.
- Stat & công dụng của part tách hoàn toàn khỏi lớp render — không phụ thuộc kỹ thuật animation nào ở trên.

## 2. Kiến trúc tổng thể (C++)

Nguyên tắc chủ đạo: **composition over inheritance** + **data-driven design**. Character lắp ghép từ Component, dữ liệu từng part nằm trong Data Asset để designer thêm part mới không cần sửa C++.

```
ACyborgCharacter (ACharacter)
 ├─ USkeletalMeshComponent* RootBodyMesh      // leader/skeleton chủ
 ├─ UCyborgPartManagerComponent               // Facade quản lý equip/swap part (dạng cây)
 ├─ UCyborgAttributeComponent / UAbilitySystemComponent (GAS)
 └─ UCyborgMeshLODComponent                   // quyết định modular-live vs merged mesh
```

### 2.1 `UCyborgPartDataAsset` (UPrimaryDataAsset)

Nguồn sự thật cho 1 loại part — mesh, stat modifier, ability cấp thêm, có cần animation riêng không, và các sub-slot mà part này cung cấp.

```cpp
UENUM(BlueprintType)
enum class ECyborgPartSlot : uint8 { Head, Torso, LeftArm, RightArm, LeftLeg, RightLeg };
// Chỉ dùng cho vùng cơ thể GỐC, luôn tồn tại trên mọi character (do skeleton quyết định).

USTRUCT(BlueprintType)
struct FCyborgStatModifier
{
    GENERATED_BODY()
    UPROPERTY(EditDefaultsOnly) FGameplayAttribute Attribute; // nếu dùng GAS
    UPROPERTY(EditDefaultsOnly) float Value = 0.f;
};

USTRUCT(BlueprintType)
struct FCyborgAttachmentSlotDefinition
{
    GENERATED_BODY()
    UPROPERTY(EditDefaultsOnly) FGameplayTag SlotTag;        // Cyborg.Slot.Torso.SidePanel.Left
    UPROPERTY(EditDefaultsOnly) FName AttachSocketName;
    UPROPERTY(EditDefaultsOnly) FGameplayTagQuery AllowedModuleQuery; // part nào được gắn vào đây
    UPROPERTY(EditDefaultsOnly) bool bIsSkinnedPart = false;  // xem mục 2.4
};

UCLASS()
class UCyborgPartDataAsset : public UPrimaryDataAsset
{
    GENERATED_BODY()
public:
    UPROPERTY(EditDefaultsOnly) ECyborgPartSlot Slot;
    UPROPERTY(EditDefaultsOnly) TSoftObjectPtr<USkeletalMesh> Mesh;
    UPROPERTY(EditDefaultsOnly) TArray<FCyborgStatModifier> StatModifiers;
    UPROPERTY(EditDefaultsOnly) TArray<TSubclassOf<UGameplayAbility>> GrantedAbilities;
    UPROPERTY(EditDefaultsOnly) bool bRequiresCustomAnimation = false;
    UPROPERTY(EditDefaultsOnly) TSubclassOf<UAnimInstance> OverrideAnimClass;
    UPROPERTY(EditDefaultsOnly) TArray<FCyborgAttachmentSlotDefinition> ProvidedSlots; // sub-slot mà part này mở ra
};
```

Dùng `TSoftObjectPtr` để không load hết mesh khi chưa cần — load qua `UAssetManager` khi thực sự equip.

### 2.2 `UCyborgPartManagerComponent` — Facade/Mediator, quản lý dạng cây

Không dùng map phẳng `TMap<ECyborgPartSlot, ...>` cho toàn hệ thống, vì sub-slot của một part (VD: Torso loại A có 2 chỗ gắn vai, Torso loại B có 4 chỗ) là thuộc tính riêng của part đó, không phải hằng số toàn cục. Nếu hardcode hết vào enum sẽ nổ tổ hợp và phá vỡ tính data-driven.

```cpp
struct FEquippedPartNode
{
    UCyborgPartDataAsset* PartAsset;
    USceneComponent* MeshComp; // USkeletalMeshComponent hoặc UStaticMeshComponent, xem 2.4
    TMap<FGameplayTag /*SlotTag*/, FEquippedPartNode> Children;
};

UCLASS()
class UCyborgPartManagerComponent : public UActorComponent
{
    GENERATED_BODY()
public:
    void EquipPart(FGameplayTag SlotTag, UCyborgPartDataAsset* NewPart);
    void RemovePart(FGameplayTag SlotTag);

    DECLARE_MULTICAST_DELEGATE_OneParam(FOnPartChanged, FGameplayTag);
    FOnPartChanged OnPartChanged; // Observer — attribute component, VFX, UI lắng nghe

private:
    TMap<FGameplayTag, FEquippedPartNode> RootParts; // Head/Torso/Arms/Legs quy về tag gốc
};
```

`EquipPart` duyệt cây theo `SlotTag`, kiểm tra `AllowedModuleQuery` trước khi cho gắn, rồi node đó tự expose thêm `ProvidedSlots` của part mới (một part vừa được gắn, vừa có thể cung cấp thêm slot cho part khác — đúng bản chất modular cyborg).

Logic chọn animation khi equip:
- `bRequiresCustomAnimation == false` → `PartMesh->SetLeaderPoseComponent(RootBodyMesh)`.
- `true` → gán `OverrideAnimClass`, AnimBP của part dùng **Copy Pose from Mesh** (source = RootBodyMesh) hoặc **Animation Layer Interface** (Linked Anim Graph) rồi blend thêm animation riêng.

### 2.3 Stat & ability — Observer, khuyến nghị GAS

Vì có cả "chỉ số" và "công dụng" (VD: tay tạo grappling hook, chân tăng tốc chạy) — đúng use case của **Gameplay Ability System**: mỗi part cấp `GameplayEffect` (modify AttributeSet) và/hoặc `GameplayAbility` khi equip, gỡ khi unequip. GAS có sẵn replication + prediction.

```cpp
void UCyborgAttributeComponent::HandlePartChanged(FGameplayTag SlotTag)
{
    // Remove effect cũ của slot, apply GameplayEffect/Ability mới từ PartAsset
}
```

Có thể bắt đầu bằng `TMap<FName, float>` tự cộng dồn nếu muốn đơn giản trước, nhưng nên giữ interface tương tự để sau migrate sang GAS không phải viết lại PartManager.

### 2.4 Phân biệt Skinned part vs Rigid attachment

- **Skinned part** (`bIsSkinnedPart = true`): phải deform theo skeleton (giáp che thân, phải theo animation chạy/nhảy) → `USkeletalMeshComponent`, set Leader Pose từ `RootBodyMesh`.
- **Rigid attachment** (`bIsSkinnedPart = false`): chỉ gắn cứng vào 1 socket, không cần deform theo bone khác (mod chip, đèn pin, module vũ khí nhỏ) → `UStaticMeshComponent` hoặc SkeletalMesh đơn giản, chỉ `AttachToComponent(ParentPartMesh, RelativeTransform, SocketName)` — không cần Leader Pose/Copy Pose, nhẹ hơn nhiều.

Việc quy định `bIsSkinnedPart` trong Data Asset giúp PartManager tự quyết định đường xử lý mà không cần if/else theo tên slot cụ thể.

### 2.5 Tối ưu hiển thị — Factory + Cache

```cpp
UCLASS()
class UCyborgMeshMergeSubsystem : public UWorldSubsystem
{
    GENERATED_BODY()
public:
    USkeletalMesh* GetOrCreateMergedMesh(const TArray<UCyborgPartDataAsset*>& Combo);
private:
    TMap<uint32 /*hash tổ hợp part*/, USkeletalMesh*> MergedMeshCache;
};
```

Dùng cho NPC (bake 1 lần lúc spawn) và cho player ở xa/LOD thấp trong multiplayer — chuyển từ nhiều `SkeletalMeshComponent` sống sang 1 mesh gộp qua `FSkeletalMeshMerge` (API C++ engine, cần tự viết Function Library wrap vì không có sẵn ở Blueprint). `UCyborgMeshLODComponent` theo dõi khoảng cách/significance để quyết định gọi merge hay giữ modular sống.

### 2.6 Replication (multiplayer)

Chỉ replicate **ID/reference của part** (VD: `TArray<TSoftObjectPtr<UCyborgPartDataAsset>>` theo slot tag), không replicate mesh. Dùng `RepNotify` để client tự gọi lại `EquipPart` local khi giá trị thay đổi — băng thông rẻ, mỗi client tự load asset qua AssetManager.

## 3. Design pattern áp dụng

| Pattern | Vai trò |
|---|---|
| Data-driven / Strategy | Hành vi & stat của part nằm trong Data Asset, không hardcode theo class |
| Facade/Mediator | `PartManagerComponent` là cửa duy nhất để equip/remove, ẩn logic mesh + anim mode |
| Composite | Part có thể tự cung cấp sub-slot cho part khác → cây equip đệ quy, không phải map phẳng |
| Observer | Delegate `OnPartChanged` decouple stat system, VFX, UI khỏi PartManager |
| Factory + Cache | `MeshMergeSubsystem` tạo & cache mesh gộp theo hash tổ hợp part |
| Component composition | Character lắp từ component độc lập thay vì kế thừa sâu theo loại cyborg |

## 4. Các bước tiến hành đề xuất

1. Định nghĩa `ECyborgPartSlot` (root slot), `FGameplayTag` cho sub-slot, `FCyborgStatModifier`, `FCyborgAttachmentSlotDefinition`, `UCyborgPartDataAsset`.
2. Dựng skeleton chung + base AnimBP chạy trên `RootBodyMesh`.
3. Viết `UCyborgPartManagerComponent` dạng cây: equip/remove theo `SlotTag`, Leader Pose wiring, delegate.
4. Với part cần animation riêng: chọn Copy Pose node hoặc Animation Layer Interface, test riêng 1–2 part mẫu trước khi làm hàng loạt.
5. Chọn hướng stat: GAS (khuyến nghị nếu có ability đặc biệt) hoặc custom AttributeComponent nhẹ, giữ interface tương tự để dễ migrate.
6. Viết `MeshMergeSubsystem` + hash combo, chỉ áp dụng cho NPC/LOD xa trước, chưa vội optimize cho player chính.
7. Thiết lập replication cho multiplayer (chỉ id part theo slot tag, RepNotify).
8. Benchmark draw call với `stat unit`, `stat gpu` khi nhiều cyborg modular hiển thị cùng lúc để xác định ngưỡng khi nào cần chuyển sang merged mesh.
