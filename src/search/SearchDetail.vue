<style>
@import "./textstyle.css";
</style>
<template>
  <div style="color: gray">
    <v-row>
      <v-col cols="8" style="margin-left: auto; margin-right: auto">
        <v-card class="mx-auto" style="padding: 20px">
          <v-img
            :src="recipe.image"
            max-height="400px"
            alt="Main Image"
          ></v-img>

          <div style="margin-top: 5px; display: flex; float: right">
            <v-card-text style="flex: 20"
              >작성자: {{ recipe.userNickName }}</v-card-text
            >
          </div>
          <v-card-title
            style="
              font-family: ELAND_Nice_M;
              white-space: nowrap;
              overflow: hidden;
              text-overflow: ellipsis;
            "
          >
            {{ recipe.recipeName }}
          </v-card-title>
          <v-card-subtitle> {{ recipe.tip }} </v-card-subtitle>
          <v-expand-transition>
            <div>
              <v-divider></v-divider>
              <v-card-text> {{ recipe.explanation }} </v-card-text>
            </div>
          </v-expand-transition>
        </v-card>
        <v-card style="margin-top: 30px">
          <v-card-title style="font-weight: bold">재료</v-card-title>
          <v-divider></v-divider>
          <v-row align="start" justify="center">
            <v-col cols="12" md="4" v-for="(items, i) in stuffs" :key="i">
              <v-simple-table dense>
                <thead style="background-color: blue">
                  <tr>
                    <th class="white--text">Stuff</th>
                    <th class="white--text">Quentity</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(item, j) in items" :key="j">
                    <td>{{ item.stuffName }}</td>
                    <td>{{ item.quantity }}</td>
                  </tr>
                </tbody>
              </v-simple-table>
            </v-col>
          </v-row>
        </v-card>

        <v-card
          style="margin-top: 30px; font-weight: bold; font-family: yg-jalnan"
          v-for="(item, i) in recipe.recipeProcedure"
          :key="i"
        >
          <v-row align="center" justify="center">
            <v-col cols="12" md="8">
              <v-card-text style="font-size: 15pt">
                {{ item.recipeProcedure }}
              </v-card-text>
            </v-col>
            <v-col cols="12" md="4">
              <v-img
                :src="item.recipeProcedureImage"
                width="350"
                alt="Recipe Procedure Image"
              />
            </v-col>
          </v-row>
        </v-card>

        <v-card style="margin-top: 30px">
          <v-card-title style="font-weight: bold">판매 재료</v-card-title>
          <v-divider></v-divider>
          <v-row>
            <v-col v-for="(item, i) in products" :key="i" cols="12" md="3">
              <v-card
                style="text-align: center; margin: 10px"
                @click="navigateToMarket(item.id)"
                flat
              >
                <v-img
                  :src="item.productTitleImage"
                  height="150px"
                  min-width="200px"
                  alt="Market Stuff Image"
                />
                <v-list-item-title style="margin: 10px; font-weight: bold">{{
                  item.name
                }}</v-list-item-title>
              </v-card>
            </v-col>
          </v-row>
        </v-card>
        <v-card style="margin-top: 30px">
          <v-card-title style="font-weight: bold">추천 강의</v-card-title>
          <v-divider></v-divider>
          <v-row>
            <v-col v-for="(item, i) in lectures" :key="i" cols="12" md="3">
              <v-card
                style="text-align: center; margin: 10px"
                @click="navigateToLecture(item.id)"
                flat
              >
                <v-img
                  :src="`https://img.youtube.com/vi/${item.imageSrc}/0.jpg`"
                  height="150px"
                  min-width="200px"
                  alt="Lecture Image"
                />
                <v-list-item-title
                  style="text-align: center; margin: 10px; font-weight: bold"
                  >{{ item.title }}</v-list-item-title
                >
              </v-card>
            </v-col>
          </v-row>
        </v-card>
      </v-col>
    </v-row>
  </div>
</template>
<script>
import api from "@/api/Search";
export default {
  data: () => ({
    show: false,
    recipe: [],
    stuffs: [],
    products: [],
    lectures: [],
  }),
  mounted() {
    this.getRecipeData();
    // this.getLectureData();
    // this.getProductData();
  },
  methods: {
    async getRecipeData() {
      const id = this.$route.query.id;
      const result = await api.detail(id);
      if (result.status == 200) {
        this.recipe = result.data;
        // 'https://3.bp.blogspot.com/-ZKBbW7TmQD4/U6P_DTbE2MI/AAAAAAAADjg/wdhBRyLv5e8/s1600/noimg.gif'
        if (
          this.recipe.image == null ||
          this.recipe.image ==
            "https://3.bp.blogspot.com/-ZKBbW7TmQD4/U6P_DTbE2MI/AAAAAAAADjg/wdhBRyLv5e8/s1600/noimg.gif"
        )
          this.recipe.image = this.recipe.recipefile[0].dataUrl;

        this.sliceList();
        this.getProductData();
        this.getLectureData();
      }
    },
    navigateToLecture(lectureId) {
      this.$router.push(`/lecture/detail/${lectureId}`);
    },
    navigateToMarket(productId) {
      this.$router.push({
        name: "ProductDetail",
        params: { id: productId },
      });
    },
    sliceList() {
      let i = this.recipe.stuffRecipe.length / 3;

      this.stuffs[0] = this.recipe.stuffRecipe.slice(0, i + 1);

      this.stuffs[1] = this.recipe.stuffRecipe.slice(i + 1, i * 2 + 1);

      this.stuffs[2] = this.recipe.stuffRecipe.slice(i * 2 + 1);
    },
    async getLectureData() {
      if (this.recipe.category == null) this.recipe.category = " ";

      const results = await api.lectureList(this.recipe.category);
      if (results.status == 200) {
        this.lectures = results.data;
      }
    },
    async getProductData() {
      const results = await api.productList(this.recipe.stuffRecipe);
      if (results.status == 200) {
        this.products = results.data;
      }
    },
  },
};
</script>