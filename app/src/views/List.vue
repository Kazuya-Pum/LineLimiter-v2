<template>
  <v-container fluid class="pa-0">
    <v-dialog
      v-model="dialog"
      fullscreen
      scrollable
      hide-overlay
      transition="dialog-bottom-transition"
    >
      <food-details
        :foodId="foodId"
        :enabled="enabled"
        @close="dialog = false"
      />
    </v-dialog>
    <v-toolbar dark style="background-color: #46465a">
      <v-text-field
        prepend-icon="mdi-magnify"
        single-line
        hide-details
        v-model="search"
      ></v-text-field>
    </v-toolbar>
    <v-tabs icons-and-text grow show-arrows center-active v-model="tab">
      <v-tab>すべて<v-icon large>$vuetify.icons.all</v-icon></v-tab>
      <v-tab>生鮮<v-icon large>mdi-food-apple</v-icon></v-tab>
      <v-tab>調味料<v-icon large>mdi-shaker</v-icon></v-tab>
      <v-tab>保存食<v-icon large>$vuetify.icons.preserved</v-icon></v-tab>
      <v-tab>その他<v-icon large>$vuetify.icons.other</v-icon></v-tab>
    </v-tabs>
    <v-card flat class="overflow-y-auto pa-0 text-center">
      <v-tabs-items v-model="tab">
        <v-tab-item>
          <food-list
            :enabled="enabled"
            :search="search"
            @click="onClick"
          ></food-list>
        </v-tab-item>
        <v-tab-item>
          <food-list
            category="生鮮食品"
            :enabled="enabled"
            :search="search"
            @click="onClick"
          ></food-list>
        </v-tab-item>
        <v-tab-item>
          <food-list
            category="調味料"
            :enabled="enabled"
            :search="search"
            @click="onClick"
          ></food-list>
        </v-tab-item>
        <v-tab-item>
          <food-list
            category="保存食"
            :enabled="enabled"
            :search="search"
            @click="onClick"
          ></food-list>
        </v-tab-item>
        <v-tab-item>
          <food-list
            category="その他"
            :enabled="enabled"
            :search="search"
            @click="onClick"
          ></food-list>
        </v-tab-item>
      </v-tabs-items>
    </v-card>
  </v-container>
</template>

<script lang="ts">
import Vue from "vue";
import FoodDetails from "../components/FoodDetails.vue";
import FoodList from "../components/FoodList.vue";

export default Vue.extend({
  components: {
    FoodDetails,
    FoodList,
  },
  props: {
    enabled: {
      type: Boolean,
      default: true,
    },
  },
  data: () => ({
    tab: 0,
    dialog: false,
    foodId: "",
    search: "",
  }),
  methods: {
    onClick(id: string) {
      this.dialog = true;
      this.foodId = id;
    },
  },
});
</script>
