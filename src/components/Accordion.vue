<template>
<v-container>
    <v-row>
        <v-col cols="3">
        <v-expansion-panels accordion>
            <v-expansion-panel v-for="(item,i) in 5" :key="i">
            <v-expansion-panel-header :id="'category-'+i">Item</v-expansion-panel-header>
            <v-expansion-panel-content :id="'category-'+i+'title'">Lorem ipsum.</v-expansion-panel-content>              
            </v-expansion-panel>
        </v-expansion-panels>
        <v-btn>new</v-btn>
        </v-col>
    </v-row>
</v-container>
</template>

<script>
export default {
    
}
</script>