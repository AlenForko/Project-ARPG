// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "Abilities/GameplayAbility.h"
#include "FireballAbility.generated.h"

class AProjectile;
/**
 * 
 */
UCLASS()
class ARPG_AKC_API UFireballAbility : public UGameplayAbility
{
	GENERATED_BODY()

	UFireballAbility();

protected:
	virtual void ActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, const FGameplayEventData* TriggerEventData) override;

	void EndThisAbility(bool wasCanceled);

protected:
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Fireball")
	TSubclassOf<AProjectile> FireballActor;

	UFUNCTION()
	void FireballHit(FGameplayEventData data);
	
};
