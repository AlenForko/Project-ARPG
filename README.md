# Graduation Project: Advanced Loot System in Unreal Engine 5.3



## Table of Contents
- [Trailer](https://www.youtube.com/watch?v=q3CqZs5tK4k)
- [Project Overview](#project-overview)
- [Features](#features)
- [Screenshots](#screenshots)

## Project Overview
This project is my graduation project developed in Unreal Engine 5.3 using C++. It focuses on systematic programming and user interface (UI) design. The project includes a comprehensive loot system, item generation system, and an equipment and inventory system, all with corresponding UIs.

## Features
### **Loot System**: [LevelTileManager.cpp 149 - 201](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/Private/LevelTileManager.cpp) 
> [!NOTE]
> - The "**LevelTileManager**" was initially designed to manage both world tile generation and loot generation. After merging functionalities, the name LevelTileManager remained.
> - The loot system provides dynamic and configurable loot drops based on predefined parameters and player actions. This ensures that the loot remains relevant and exciting for players, with variations in rarity and item types. 

<details open>  
  
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

### **Item Generation System**: [DataLibrary.h](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/DataLibrary.h)
  
> [!NOTE]
> - [DataLibrary.h](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/DataLibrary.h) is then used to generate an item based on stats which are created in a Data Asset and then added to a Data Table which is meant to be a loot table. Once added to the loot table, the level manager uses that table to roll for X amount of items to drop based on their weights.
> - The item generation system creates items with randomized attributes and properties, ensuring a wide variety of possible items. Each generated item can have different stats, effects, and rarity, adding depth to the gameplay and replayability.

### **Affixes Generation System**: [AffixLibrary.h](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/Public/AffixLibrary.h) [AffixLibrary.cpp](https://github.com/AlenForko/Project-ARPG/blob/main/Source/ARPG_AKC/Private/AffixLibrary.cpp)
- Randomized Prefixes & Suffixes for items based on item rarity.

### **Equipment System**:
- The equipment system allows players to manage and equip items on their characters. Each item comes with detailed stats that affect the character's performance, such as damage, defense, and special abilities. This system provides a strategic layer to the game, as players must choose the best equipment for their play style.

### **Inventory System**:
- The inventory system provides a user-friendly interface for organizing and interacting with items. Players can sort, use, or discard items easily, and the inventory UI offers a clear view of the items' details and their effects on the character.

### **User Interface**:
- The user interface is designed to be intuitive and responsive, providing a seamless experience for players. Each system's UI (loot, item generation, equipment, and inventory) is crafted to be easy to navigate and understand, enhancing the overall user experience.

## Screenshots
![Loot System UI](path-to-loot-system-ui-screenshot)
![Inventory System UI](path-to-inventory-system-ui-screenshot)
![Equipment System UI](path-to-equipment-system-ui-screenshot)
