<template>
  <v-container fluid class="pa-0" style="height: 100%">
    <v-form style="background-color: #eeeeee; height: 100%">
      <div
        class="d-flex align-stretch flex-column flex-sm-row"
        style="height: 100%; max-height: calc(100vh - 56px)"
      >
        <v-card
          flat
          height="100vmin"
          min-height="20vmin"
          min-width="50vmin"
          style="background-color: #eeeeee"
          class="flex-grow-0 flex-shrink-1"
        >
          <label
            for="file_img"
            style="
              display: block;
              height: 100%;
              background-position: center;
              background-repeat: no-repeat;
              background-size: 30%;
            "
            :style="{ backgroundImage }"
          >
            <v-file-input
              name="file"
              id="file_img"
              accept="image/*"
              class="d-none"
              v-model="preview"
            />
            <v-img v-if="url" :src="url" height="100%">
              <template v-slot:placeholder>
                <v-row class="fill-height ma-0" align="center" justify="center">
                  <v-progress-circular
                    indeterminate
                    color="grey lighten-5"
                  ></v-progress-circular>
                </v-row>
              </template>
            </v-img>
          </label>
        </v-card>
        <div class="flex-grow-1 flex-shrink-0">
          <v-card class="rounded-t-xl overflow-y-auto" height="100%">
            <v-card-title>
              <v-text-field
                v-model.trim="food.name"
                label="食品名を入力"
              ></v-text-field>
            </v-card-title>
            <v-tabs icons-and-text v-model="tab" grow show-arrows center-active>
              <v-tab @click="open">
                期限
                <v-icon x-large>mdi-bell</v-icon>
              </v-tab>
              <v-tab @click="open">
                保存場所
                <v-icon x-large>mdi-fridge</v-icon>
              </v-tab>
              <v-tab @click="open">
                カテゴリ
                <v-icon x-large>mdi-shape</v-icon>
              </v-tab>
              <v-tab @click="open">
                メモ
                <v-icon x-large>mdi-file-document-edit</v-icon>
              </v-tab>
            </v-tabs>
            <v-expand-transition>
              <v-tabs-items v-model="tab" @change="open" v-show="show">
                <v-tab-item>
                  <v-card flat>
                    <v-card-text>
                      <v-dialog
                        ref="dialog"
                        v-model="modal"
                        :return-value.sync="food.date"
                        width="290px"
                      >
                        <template v-slot:activator="{ on, attrs }">
                          <v-text-field
                            v-model="food.date"
                            label="賞味期限"
                            prepend-icon="mdi-calendar"
                            readonly
                            v-bind="attrs"
                            v-on="on"
                          ></v-text-field>
                        </template>
                        <v-date-picker
                          v-model="food.date"
                          @input="$refs.dialog.save(food.date)"
                        ></v-date-picker>
                      </v-dialog>
                    </v-card-text>
                    <v-divider></v-divider>
                    <v-card-text>
                      <h3>通知日</h3>
                      <v-chip-group
                        multiple
                        v-model="food.notifications"
                        color="primary"
                      >
                        <v-chip filter :value="1">1日前</v-chip>
                        <v-chip filter :value="3">3日前</v-chip>
                        <v-chip filter :value="7">7日前</v-chip>
                        <v-chip filter :value="14">14日前</v-chip>
                      </v-chip-group>
                    </v-card-text>
                  </v-card>
                </v-tab-item>
                <v-tab-item>
                  <v-card flat>
                    <v-card-text>
                      <v-text-field
                        v-model.trim="food.place"
                        label="保存場所を入力"
                      ></v-text-field>
                      <v-chip-group v-model="food.place" color="primary">
                        <v-chip value="冷蔵庫">冷蔵庫</v-chip>
                        <v-chip value="冷凍庫">冷凍庫</v-chip>
                        <v-chip value="常温">常温</v-chip>
                      </v-chip-group>
                    </v-card-text>
                  </v-card>
                </v-tab-item>
                <v-tab-item>
                  <v-card flat>
                    <v-card-text>
                      <v-text-field
                        v-model.trim="food.category"
                        label="カテゴリを入力"
                      ></v-text-field>
                      <v-chip-group v-model="food.category" color="primary">
                        <v-chip value="生鮮食品">生鮮食品</v-chip>
                        <v-chip value="調味料">調味料</v-chip>
                        <v-chip value="保存食">保存食</v-chip>
                        <v-chip value="その他">その他</v-chip>
                      </v-chip-group>
                    </v-card-text>
                  </v-card>
                </v-tab-item>
                <v-tab-item>
                  <v-card flat>
                    <v-card-text>
                      <v-combobox
                        v-model="food.memos"
                        chips
                        clearable
                        label="メモを入力"
                        multiple
                        append-icon=""
                      >
                        <template
                          v-slot:selection="{ attrs, item, select, selected }"
                        >
                          <v-chip
                            v-bind="attrs"
                            :input-value="selected"
                            close
                            @click="select"
                            @click:close="remove(item)"
                            class="overflow-x-auto"
                          >
                            <strong>{{ item }}</strong>
                          </v-chip>
                        </template>
                      </v-combobox>
                    </v-card-text>
                  </v-card>
                </v-tab-item>
              </v-tabs-items>
            </v-expand-transition>
            <v-card-actions class="justify-center pa-3">
              <v-btn
                id="submit"
                color="primary"
                outlined
                rounded
                large
                min-width="8em"
                class="font-weight-bold"
                @click="save"
                :loading="loading"
              >
                <span>保存</span>
              </v-btn>
            </v-card-actions>
          </v-card>
        </div>
      </div>
    </v-form>
  </v-container>
</template>

<script lang="ts">
import Vue from "vue";
import { mapActions } from "vuex";
import Food from "@/types/food";
import firebase from "firebase/app";
import "firebase/storage";
import imageCompression from "browser-image-compression";

const getNow = () => {
  return new Date(Date.now() - new Date().getTimezoneOffset() * 60000)
    .toISOString()
    .substr(0, 10);
};

// アップロードされた画像ファイルを取得
const getCompressImageFileAsync = async (file: File) => {
  const options = {
    maxSizeMB: 1, // 最大ファイルサイズ
    maxWidthOrHeight: 800, // 最大画像幅もしくは高さ
  };
  try {
    // 圧縮画像の生成
    return await imageCompression(file, options);
  } catch (error) {
    console.error("getCompressImageFileAsync is error", error);
    throw error;
  }
};

const init: Food = {
  name: "",
  limit: 0,
  date: getNow(),
  notifications: [],
  place: "冷蔵庫",
  category: "",
  memos: [] as Array<string>,
  enabled: true,
  img: "",
};

export default Vue.extend({
  props: {
    foodId: {
      type: String,
      required: false,
    },
  },
  data: () => ({
    preview: null as null | File,
    backgroundImage: "url(" + require("../assets/img.png") + ")",
    imgHeight: "100vmin",
    tab: null,
    modal: false,
    show: false,
    loading: false,
    food: { ...init },
    tmpId: "",
    url: "",
  }),
  methods: {
    ...mapActions(["addFood", "updateFood"]),
    open() {
      this.show = true;
    },
    remove(item: string) {
      this.food.memos.splice(this.food.memos.indexOf(item), 1);
      this.food.memos = [...this.food.memos];
    },
    async save() {
      this.loading = true;
      this.food.limit = Date.parse(this.food.date || getNow());

      this.food.notifications = this.food.notifications.map((notify) =>
        Number(notify)
      );

      if (this.foodId) {
        this.food.enabled = true;
        await this.updateFood({ foodId: this.foodId, food: this.food });
      } else {
        const added = await this.addFood(this.food);
        this.food.id = added.id;
      }

      if (this.preview) {
        const imageUrl = await this.uploadImage(this.preview);

        this.food.img = imageUrl;
        await this.updateFood({ foodId: this.food.id, food: this.food });
      }

      this.loading = false;
      this.$router.push({ name: "List" });
    },
    getImagePath() {
      const storageId = this.$store.getters.storageId;
      const imagePath = `${storageId}/${this.food.id}`;

      return imagePath;
    },
    async uploadImage(file: File) {
      const imagePath = this.getImagePath();
      const pfile = await getCompressImageFileAsync(file);

      await firebase
        .storage()
        .ref()
        .child(imagePath)
        .put(pfile, { contentType: "image/*" });

      return firebase.storage().ref().child(imagePath).getDownloadURL();
    },
  },
  watch: {
    foodOrigin(food) {
      this.food = { ...food };

      this.food.id = this.foodId;

      this.url = this.food.img || this.url;
    },
    preview(value) {
      if (value == null) {
        return "";
      }
      this.url = URL.createObjectURL(value);
    },
  },
  computed: {
    foodOrigin(): Food {
      if (this.tmpId) {
        const food: Food = { ...this.$store.getters.foodById(this.foodId) };
        if (!food || Object.keys(food).length == 0) {
          return { ...init };
        }

        food.date = new Date(food.limit).toISOString().substr(0, 10);

        return food;
      } else {
        return { ...init };
      }
    },
  },
  mounted() {
    // 初回読み込み時にcomputedを発火させる用
    this.tmpId = this.foodId;
  },
});
</script>
