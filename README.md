# Graduation Project: Project Action Role-Playing Game

# Table of Contents
- [Trailer](https://www.youtube.com/watch?v=q3CqZs5tK4k)
- [Project Overview](#project-overview)
- [Features](#features)
- [Screenshots](#screenshots)

# Project Overview
This project is my graduation project developed in Unreal Engine 5.3 using C++. It focuses on systematic programming and user interface (UI) design. The project includes a comprehensive loot system, item generation system, and an equipment and inventory system, all with corresponding UIs.

# Features
## **Loot System**: [LevelTileManager.cpp 149 - 201](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/Private/LevelTileManager.cpp) 
> [!NOTE]
> - The "**LevelTileManager**" was initially designed to manage both world tile generation and loot generation. After merging functionalities, the name LevelTileManager remained.

> [!TIP]
> 
> <details>  
> <summary>CLICK TO REVEAL CODE -----></summary>
> 
> ```c++
>  void ALevelTileManager::GenerateLoot(ABaseEnemy* Enemy)
> {
>     FVector EnemyLocation = Enemy->GetActorLocation();
> 
>     if (!LootTable) return;
>     const FString ContextString(TEXT("Loot Table Context"));
>     const TArray<FName> RowNames = LootTable->GetRowNames();
>     if (RowNames.Num() == 0) return;
> 
>     float TotalWeight = 0;
>     for (const FName RowName : RowNames)
>     {
>         if (const FLootTable* LootRow = LootTable->FindRow<FLootTable>(RowName, ContextString))
>         {
>             TotalWeight += LootRow->Weight;
>         }
>     }
> 
>     for (int i = 0; i < NumItemsToSpawn;)
>     {
>         const float RandomWeight = FMath::FRandRange(0, TotalWeight);
> 
>         UE_LOG(LogTemp, Warning, TEXT("RandomWeight in loot: %f"), RandomWeight);
> 
>         float CurrentWeight = 0;
> 
>         for (const FName RowName : RowNames)
>         {
>             if (const FLootTable* LootRow = LootTable->FindRow<FLootTable>(RowName, ContextString))
>             {
>                 CurrentWeight += LootRow->Weight;
> 
>                 if (CurrentWeight > RandomWeight)
>                 {
>                     if (!IsValid(LootRow->ItemDataAsset)) // No loot row taken into consideration
>                     {
>                         i++;
>                         break;
>                     }
> 
>                     if (UItemDataAsset* SelectedItem = LootRow->ItemDataAsset)
>                     {
>                         SpawnLootAroundEnemy(SelectedItem, i, EnemyLocation);
>                         break;
>                     }
>                     break;
>                 }
>             }
>         }
>     }
> }
> 
> void ALevelTileManager::SpawnLootAroundEnemy(UItemDataAsset* ItemData, int32& NextItem, const FVector EnemyLocation)
> {
>     const FVector SpawnLocation = FVector(EnemyLocation.X, EnemyLocation.Y, GetActorLocation().Z);
> 
>     if (ItemData && BaseItemActor)
>     {
>         FItemGenericInfo ItemInfo = UDataLibrary::GenerateItem(ItemData->ItemGenericInfo, NextItem);
> 
>         if (ABaseItem* NewItem = GetWorld()->SpawnActor<ABaseItem>(BaseItemActor, SpawnLocation, FRotator::ZeroRotator))
>         {
>             NewItem->GenerateItem(ItemInfo);
>         }
>     }
> }
> ```
> 
> </details>

### **Overview**:
- The loot system in the project is designed to dynamically generate and manage loot drops based on predefined parameters and player actions. It ensures that the loot remains relevant and exciting for players by incorporating variations in rarity and item types.

### **Key Components**:
- **Loot Table**:
	- The loot table is a **Data Table** that contains different items in the form of **Data Assets** and their respective weights.

- **Weight Calculation**:
	- The system calculates the total weight of all items in the loot table. This total weight is used to determine the probability of each item being dropped.

- **Loot Generation**:
	- The GenerateLoot function is responsible for generating loot when an enemy is defeated.
	- It takes the enemy’s location as input and uses the loot table to randomly select items based on their weights.
	- The function ensures that the number of items spawned (**NumItemsToSpawn**) is managed through rarity and that the loot is distributed around the enemy’s location.

### **Loot Table example**:

![Loot Table example](https://github.com/AlenForko/Project-ARPG/blob/main/Screenshots/LootTable.png)

### **Loot Explosion**:
![Loot Explosion](https://github.com/AlenForko/Project-ARPG/blob/main/Screenshots/Showcase.png)

## **Item Generation System**: [DataLibrary.h](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/DataLibrary.h) | [BaseItem.h](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/BaseItem.h) | [BaseItem.cpp](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/BaseItem.cpp)
  
### **Overview**:
- [DataLibrary.h](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/DataLibrary.h) defines several enumerations, structs, and inline functions that are fundamental for the item generation system in our project. These definitions help in creating and managing item attributes and their behaviors, which are then used in data assets and loot tables.

### **Key Components**:
- **Enumerations (UENUM)**:
	- EItemSlotType: Defines different item slots (WeaponSlot, NecklaceSlot, etc.).
	- EItemRarity: Specifies item rarities with display names and weight values (Common, Uncommon, Rare, Unique).
	- EItemType: Lists types of items (Gold, Weapon, Armor, etc.).
	- EColorType: Defines color types used to represent item rarities (White, Blue, Yellow, Orange).
 
- **Structs (USTRUCT)**:
	- FItemTextData: Contains textual data for items such as name, description, and interaction text.
	- FItemStats: Holds item statistics like damage, armor, consumable value, and item type.
	- FItemNumericData: Manages stackable properties of items, including max stack size.
	- FItemGenericInfo: A comprehensive struct that encapsulates all item-related data, including slot type, rarity, color, text data, stats, numeric data, icon, mesh, and particle effects.

- **Inline Functions**:
	- GenerateRandomRarity: Generates a random rarity for an item based on defined weights.
	- SetColor: Sets the item's color based on its rarity.
	- SetParticleEffect: Sets the particle effect associated with the item rarity.
	- GetMinMaxAffixes: Returns the minimum and maximum number of affixes based on item rarity.

 - **Example Integration**:
	- [DataLibrary.h](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/DataLibrary.h) definitions are used in conjunction with **Data Assets** to define item properties.
	- These assets are then fed into a **Data Table**, a.k.a loot table, which the level manager uses to determine item drops during gameplay.

### **Item Generation Example**:
  
![Item Generation Example](https://github.com/AlenForko/Project-ARPG/blob/main/Screenshots/ItemExample.png)

## **Affixes Generation System**: [AffixLibrary.h](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/Public/AffixLibrary.h) | [AffixLibrary.cpp](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/Private/AffixLibrary.cpp)

### **Overview**:
The Affixes Generation System in our project dynamically enhances item attributes by applying prefixes and suffixes based on item rarity and type. This system is pivotal in offering diverse customization options and strategic depth to players, enriching gameplay through varied item attributes and effects.

### **Key Components**:
- **Enumerations (UENUM)**:
	- **EAffixType**: Defines types of affixes applied to items (Prefix, Suffix).
	- **EElementalType**: Specifies elemental attributes affected by affixes (Physical, Fire, Cold, etc.).
	- **EAttributeType**: Identifies the attribute modified by affixes (Armor, Damage, Health, etc.).
	- **EValueType**: Determines whether affix values are percentages or flat amounts (Percentage, Flat).
	- **ETextType**: Specifies the format of textual descriptions for affix effects (ValueToElementAttributeType, ValuePercentageElementAttributeType, etc.).
   
- **Structs (USTRUCT)**:
	- **FAffixesInfo**: Contains detailed data for affixes, including textual descriptions, value ranges, types, and attribute modifiers. This struct ensures consistent application of affix effects across items.

- **Functions**:
	- **ReturnTextTemplate**: Generates a formatted text description based on affix type, element type, and attribute type.
	- **GenerateAffixes**: A function that randomly selects and applies affixes to items based on specified minimum and maximum counts. Affixes are chosen from a data table and categorized into prefixes and suffixes, maintaining balance according to item rarity.

- **Example Integration**:
	- The definitions and structures from [AffixLibrary.h](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/AffixLibrary.h) are integrated with **Data Assets** to define item properties and behaviors.
	- These definitions ensure that each item can dynamically receive affixes that enhance its attributes and effects based on its rarity and type.

### **Affixes Generation Example**:
![Item Affixes Example](https://github.com/AlenForko/Project-ARPG/blob/main/Screenshots/AffixesExample.png)

> [!TIP]
> 
> <details>
> <summary>CLICK TO REVEAL GenerateAffixes function call -----></summary>
> 	
> ``` c++
> void ABaseItem::GenerateItem(FItemGenericInfo& ItemData)
> {
> 	GenerationInfo = ItemData;
> 	const float LootUpwardsImpulse = FMath::RandRange(600, 1000);
> 	
> 	// Change mesh
> 	MeshComponent->SetStaticMesh(GenerationInfo.Mesh);
> 
> 	if(!bAffixesGenerated)
> 	{
> 		// Generate Affixes
> 		AffixDataAssets = UAffixLibrary::GenerateAffixes(
> 			GenerationInfo.GetMinMaxAffixes().first,
> 			GenerationInfo.GetMinMaxAffixes().second,
> 			AffixTable);
> 	}
> 	
> 	GenerateParticleEffect();
> 	
> 	// Push it in a random direction
> 	const FVector RandomDirection = FVector(FMath::VRand()).GetSafeNormal() * LootUpwardsImpulse;
> 	
> 	BoxComponent->AddImpulse(RandomDirection);
> }
> 
> ```
> </details>

## **Inventory System**: [InventoryComponent.h](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/Public/InventoryComponent.h) | [InventoryComponent.cpp](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/Private/InventoryComponent.cpp) | [InventoryInterface.h](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/Public/InventoryInterface.h)

### **Overview**:
- The inventory system in the project is designed to manage items efficiently using a combination of the Observer and MVC (Model-View-Controller) design patterns. This system ensures a clean separation between the inventory's logic and its user interface representation, allowing for scalable and maintainable code.

### **Key Components**:
- **InventoryInterface**:
	- This interface defines the functions that must be implemented by any class that wants to observe inventory changes. It includes methods for handling item additions, removals, and movements. 

- **InventoryComponent**:
	- This class is responsible for managing the inventory logic. It handles adding, removing, and moving items, as well as notifying registered observers about these changes. The inventory is managed using a **TMap**, which is similar to a dictionary, where each slot is keyed by an integer.
   
- **Inline Functions**:
	- **NotifyItemAdded**: Notifies all registered observers that an item has been added.
	- **NotifyItemRemoved**: Notifies all registered observers that an item has been removed.
	- **NotifyItemMoved**: Notifies all registered observers that an item has been moved.

## **Inventory UI Classes**:

### **Grid Widget**: [GridWidget.h](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/Public/GridWidget.h) | [GridWidget.cpp](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/Private/GridWidget.cpp)
- The Grid Widget is designed exclusively for constructing the inventory grid and monitoring any additions or movements of items within it. The precise location of each item is efficiently calculated using an inline macro, ensuring accurate and automatic placement within the grid structure.
    
> [!TIP]
> <details>
> <summary>CLICK TO REVEAL Inline Macro -----></summary>
> 	
> ``` c++
>#define DETERMINE_GRID_LOCATION(ChildCount, MaxPerRow, Row, Column) \
>{ \
>	Row = FMath::Floor(static_cast<float>(ChildCount) / MaxPerRow); \
>	Column = ChildCount % MaxPerRow; \
>} \
> 
> ```
> </details>

### **Slot Widget**: [SlotWidget.h](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/Public/SlotWidget.h) | [SlotWidget.cpp](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/Private/SlotWidget.cpp)
- The Slot Widget serves as the visual representation for individual slots within the player's inventory and equipment UI panels. Each instance of SlotWidget represents a single slot that can hold an item, allowing for a dynamic and interactive inventory management experience. The primary purpose of the Slot Widget class is to facilitate the drag-and-drop functionality for items within the inventory and equipment systems. By doing so, it enables players to easily organize their inventory, move items between slots, and equip or unequip items from their character's equipment slots.

## **Equipment System**: [EquipmentComponent.h](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/Public/EquipmentComponent.h) | [EquipmentComponent.cpp](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/Private/EquipmentComponent.cpp) | [EquipmentInterface.h](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/Interfaces/EquipmentInterface.h)
- The Equipment System is a core feature of the project, allowing players to equip, unequip, and manage various items on their characters. This system is closely integrated with the inventory and ability systems, providing both visual representation and gameplay effects when different gear is equipped.
  
### **Key Components**:
- **Equipment Slots**:
	- The system defines various equipment slots on the character, such as Head, Weapon, Chest, Gloves, and Boots. Each slot can hold specific types of items, which are visually represented through static meshes attached to the character.

- **Equip and Unequip Functions**:
	- The EquipItem function is responsible for equipping an item, duplicating the item object, and notifying other systems (such as UI) that the item has been equipped.
	- The UnEquipItem function handles the unequipping process, removing the item's effects and visual representation from the character.
   
- **Item Affixes and Attributes**:
	- When an item is equipped, the system applies the item's affix stats (e.g., bonuses to health, damage, or resistance) to the player. This is done dynamically through the Gameplay Effect system, which ensures that the player's attributes are updated based on the equipped gear.
	- The GivePlayerAffixStats and RemoveAffixStatsFromPlayer functions manage the application and removal of these bonuses.
 
- **Observers and Delegates**:
	- The Equipment System uses an observer pattern, where different components (like UI elements) can register as observers to get notified when items are equipped or unequipped. This allows for real-time updates to the player's HUD and other interfaces.
	- Delegates such as OnGearEquipChangeDelegate and OnGearUnEquipChangeDelegate are broadcasted whenever gear changes occur, ensuring that all relevant systems are in sync.
   
### **Mesh Management**:
- The system also handles the visual representation of equipped items by attaching the appropriate mesh to the character model. For example, when a helmet is equipped, its mesh is attached to the character's head socket, and the mesh is made visible. Similarly, when the item is unequipped, the mesh is detached and hidden.
  
# Early Stage Screenshots
![Showcase1](https://github.com/AlenForko/Project-ARPG/blob/main/Screenshots/EarlyLootStage.png)
![Showcase3](https://github.com/AlenForko/Project-ARPG/blob/main/Screenshots/showcase3.png)
