<template>
  <v-container>
    <v-card :loading="isLoading">
      <v-img v-if="image" height="400" :src="image">
        <template v-slot:placeholder>
          <v-row class="fill-height ma-0" align="center" justify="center">
            <v-progress-circular indeterminate color="grey lighten-5"></v-progress-circular>
          </v-row>
        </template>
      </v-img>
      <br v-else />

      <RecipePageActionMenu
        logged-in
        :value="true"
        @json="jsonEditor = !jsonEditor"
        @edit="jsonEditor = !jsonEditor"
        @save="createRecipe"
      />

      <div v-show="jsonEditor">
        <!-- Probably not the best way, but it works! -->
        <br />
        <br />
        <VJsoneditor v-model="recipeDetails" height="1500px" :options="jsonEditorOptions" />
      </div>

      <RecipeEditor ref="recipeEditor" v-show="!jsonEditor" v-model="recipeDetails" @upload="getImage" />
    </v-card>
  </v-container>
</template>

<script>
import { api } from "@/api";

import RecipeEditor from "@/components/Recipe/RecipeEditor";
import RecipePageActionMenu from "@/components/Recipe/RecipePageActionMenu";
export default {
  components: {
    VJsoneditor: () => import(/* webpackChunkName: "json-editor" */ "v-jsoneditor"),
    RecipeEditor,
    RecipePageActionMenu,
  },
  data() {
    return {
      isLoading: false,
      fileObject: null,
      selectedFile: null,
      image: null,
      jsonEditor: false,
      jsonEditorOptions: {
        mode: "code",
        search: false,
        mainMenuBar: false,
      },
      recipeDetails: {
        name: "",
        description: "",
        image: "",
        recipeYield: "",
        recipeIngredient: [],
        recipeInstructions: [],
        slug: "",
        filePath: "",
        tags: [],
        categories: [],
        // dateAdded: "",
        notes: [],
        extras: {},
        assets: [],
        settings: {
          public: true,
          showNutrition: true,
          showAssets: true,
          landscapeView: true,
          disableComments: false,
          disableAmount: true,
        },
      },
    };
  },
  created() {
    this.getDefaultSettings();
  },

  methods: {
    async getDefaultSettings() {
      const response = await api.recipes.getDefaultSettings();
      this.recipeDetails.settings = response.data;
    },
    getImage(fileObject) {
      this.fileObject = fileObject;
      this.onFileChange();
    },
    onFileChange() {
      this.image = URL.createObjectURL(this.fileObject);
    },

    async createRecipe() {
      if (this.$refs.recipeEditor.validateRecipe()) {
        this.isLoading = true;

        if (this.fileObject) {
          this.recipeDetails.image = this.fileObject.name;
        }
        let slug = await api.recipes.create(this.recipeDetails);

        if (this.fileObject) {
          api.recipes.updateImage(slug, this.fileObject, true);
        }

        this.isLoading = false;

        this.$router.push(`/recipe/${slug}`);
      }
    },
  },
};
</script>

<style>
.img-input {
  position: absolute;
  bottom: 0;
}
</style>
