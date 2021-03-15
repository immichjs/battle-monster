<template>
    <div>
        <Attack @emitAttack="attack"/>
        <SpecialAttack @emitSpecialAttack="specialAttack"/>
        <Heal @emitHeal="heal" :disabled="isDisabled"/>
        <Surrender @emitSurrender="surrender"/>
    </div>
</template>

<script>
import Attack from './components/Attack'
import SpecialAttack from './components/SpecialAttack'
import Heal from './components/Heal'
import Surrender from './components/Surrender'

export default {
    components: {
        Attack,
        SpecialAttack,
        Heal,
        Surrender
    }
}
</script>

<style>

</style>