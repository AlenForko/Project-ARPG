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

<details>  
<summary>CLICK TO REVEAL CODE -----></summary>
	
```c++
 void ALevelTileManager::GenerateLoot(ABaseEnemy* Enemy)
{
	FVector EnemyLocation = Enemy->GetActorLocation();

	if (!LootTable) return;
	const FString ContextString(TEXT("Loot Table Context"));
	const TArray<FName> RowNames = LootTable->GetRowNames();
	if (RowNames.Num() == 0) return;

	float TotalWeight = 0;
	for (const FName RowName : RowNames)
	{
		if (const FLootTable* LootRow = LootTable->FindRow<FLootTable>(RowName, ContextString))
		{
			TotalWeight += LootRow->Weight;
		}
	}

	for (int i = 0; i < NumItemsToSpawn;)
	{
		const float RandomWeight = FMath::FRandRange(0, TotalWeight);

		UE_LOG(LogTemp, Warning, TEXT("RandomWeight in loot: %f"), RandomWeight);

		float CurrentWeight = 0;

		for (const FName RowName : RowNames)
		{
			if (const FLootTable* LootRow = LootTable->FindRow<FLootTable>(RowName, ContextString))
			{
				CurrentWeight += LootRow->Weight;

				if (CurrentWeight > RandomWeight)
				{
					if (!IsValid(LootRow->ItemDataAsset)) // No loot row taken into consideration
					{
						i++;
						break;
					}

					if (UItemDataAsset* SelectedItem = LootRow->ItemDataAsset)
					{
						SpawnLootAroundEnemy(SelectedItem, i, EnemyLocation);
						break;
					}
					break;
				}
			}
		}
	}
}

void ALevelTileManager::SpawnLootAroundEnemy(UItemDataAsset* ItemData, int32& NextItem, const FVector EnemyLocation)
{
	const FVector SpawnLocation = FVector(EnemyLocation.X, EnemyLocation.Y, GetActorLocation().Z);

	if (ItemData && BaseItemActor)
	{
		FItemGenericInfo ItemInfo = UDataLibrary::GenerateItem(ItemData->ItemGenericInfo, NextItem);

		if (ABaseItem* NewItem = GetWorld()->SpawnActor<ABaseItem>(BaseItemActor, SpawnLocation, FRotator::ZeroRotator))
		{
			NewItem->GenerateItem(ItemInfo);
		}
	}
}
```

</details>

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

// ADD IMAGE HERE OF LOOT TABLE


## **Item Generation System**: [DataLibrary.h](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/DataLibrary.h)
  
> [!NOTE]
> - It is used to generate the item information which are created in separate **Data Assets** and those are fed into a **Data Table** A.K.A loot table.
> - Once added to the loot table, the level manager uses that table to roll for X amount of items to drop based on their weights and rarity.

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
  
// ADD IMAGE HERE

## **Affixes Generation System**: [AffixLibrary.h](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/Public/AffixLibrary.h) [AffixLibrary.cpp](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/Private/AffixLibrary.cpp)
> [!NOTE]
> - Randomized Prefixes & Suffixes for items based on item rarity and stats in [BaseItem.cpp](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/BaseItem.cpp).
> - With the help of a FORCEINLINE function in [DataLibrary.h](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/Public/AffixLibrary.h), it assigns the item with the minimum and maximum amount of affixes possible to said item.

<details>  
<summary>CLICK TO REVEAL FORCEINLINE function -----></summary>
	
```c++
FORCEINLINE std::pair<int32, int32> GetMinMaxAffixes() const
	{
		int32 Min = 0;
		int32 Max = 0;
		
		switch (ItemRarity)
		{
		case EItemRarity::Common:
			break;
		case EItemRarity::Uncommon:
			Min = 1;
			Max = 2;
			break;
		case EItemRarity::Rare:
			Min = 3;
			Max = 6;
			break;
		case EItemRarity::Unique:
			break;
		}
		return std::make_pair(Min, Max);
	}
```

</details>  

<details>
<summary>CLICK TO REVEAL GenerateAffixes function call -----></summary>
	
```c++
void ABaseItem::GenerateItem(FItemGenericInfo& ItemData)
{
	GenerationInfo = ItemData;
	const float LootUpwardsImpulse = FMath::RandRange(600, 1000);
	
	// Change mesh
	MeshComponent->SetStaticMesh(GenerationInfo.Mesh);

	if(!bAffixesGenerated)
	{
		// Generate Affixes
		AffixDataAssets = UAffixLibrary::GenerateAffixes(
			GenerationInfo.GetMinMaxAffixes().first,
			GenerationInfo.GetMinMaxAffixes().second,
			AffixTable);
	}
	
	GenerateParticleEffect();
	
	// Push it in a random direction
	const FVector RandomDirection = FVector(FMath::VRand()).GetSafeNormal() * LootUpwardsImpulse;
	
	BoxComponent->AddImpulse(RandomDirection);
}

```
</details>

## **Equipment System**:
- The equipment system allows players to manage and equip items on their characters. Each item comes with detailed stats that affect the character's performance, such as damage, defense, and special abilities. This system provides a strategic layer to the game, as players must choose the best equipment for their play style.

## **Inventory System**:
- The inventory system provides a user-friendly interface for organizing and interacting with items. Players can sort, use, or discard items easily, and the inventory UI offers a clear view of the items' details and their effects on the character.

## **User Interface**:
- The user interface is designed to be intuitive and responsive, providing a seamless experience for players. Each system's UI (loot, item generation, equipment, and inventory) is crafted to be easy to navigate and understand, enhancing the overall user experience.

# Screenshots
![Loot System UI](path-to-loot-system-ui-screenshot)
![Inventory System UI](path-to-inventory-system-ui-screenshot)
![Equipment System UI](path-to-equipment-system-ui-screenshot)
