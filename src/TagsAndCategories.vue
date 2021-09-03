<template>
  <v-overlay :absolute="true">
    <v-card
      flat
      light
      :loading="loading"
      class="pa-5 rounded-0"
      color="grey lighten-5"
      id="card"
    >
      <v-row class="pl-16">
        <v-col class="pa-0 pl-16">
          <v-card color="grey lighten-2" class="d-flex justify-center text-h6">
            {{ fileDetails ? file.Name : file.name }} <br />
            Tags and Categories
          </v-card>
        </v-col>
        <v-col class="d-flex justify-end pa-0">
          <v-btn
            icon
            fab
            top
            right
            class="rounded-0"
            small
            @click="fileDetails ? forceClose() : closeWindow()"
          >
            <v-icon>mdi-close-thick</v-icon>
          </v-btn>
        </v-col>
      </v-row>

      <v-row>
        <v-card-text>
          <v-text-field
            v-model="availableFilter"
            label="Search tags and categories"
          ></v-text-field>
        </v-card-text>
      </v-row>

      <v-row>
        <v-card class="pa-2 mx-2">
          <v-row>
            <v-col>
              <v-card-text> Tags</v-card-text>
            </v-col>
            <v-col align-self="center" class="d-flex justify-end pr-5">
              <v-btn
                fab
                depressed
                icon
                small
                @click.stop="openAddTag()"
                title="add a new tag"
              >
                <v-icon color="grey lighten-1">mdi-plus-circle</v-icon>
              </v-btn>
            </v-col>
          </v-row>
          <v-divider />
          <!-- list of the tags if any exist, otherwise relevant message -->
          <v-container v-if="ownedTags && ownedTags.length > 0">
            <v-virtual-scroll
              :bench="1"
              :items="ownedTags"
              :height="listedItemsHeight"
              width="200"
              item-height="45"
            >
              <template v-slot:default="{ item }">
                <v-list-item :key="item.Id" class="customListItem">
                  <v-chip
                    color="white"
                    @click="transport(ownedTags, notOwnedTags, item)"
                  >
                    {{ item.Name }}
                  </v-chip>
                </v-list-item>
              </template>
            </v-virtual-scroll>
          </v-container>
          <v-container v-else>No attached tags</v-container>
        </v-card>

        <v-card class="pa-2 mx-2">
          <v-row>
            <v-col>
              <v-card-text> Categories</v-card-text>
            </v-col>
            <v-col align-self="center" class="d-flex justify-end pr-5">
              <v-btn
                fab
                depressed
                icon
                small
                @click.stop="openAddCategory()"
                title="add a new category"
              >
                <v-icon color="grey lighten-1">mdi-plus-circle</v-icon>
              </v-btn>
            </v-col>
          </v-row>
          <v-divider />
          <!-- list of the categories if any exist, otherwise relevant message -->
          <v-container v-if="ownedCategories && ownedCategories.length > 0">
            <v-virtual-scroll
              :bench="1"
              :items="ownedCategories"
              :height="listedItemsHeight"
              width="200"
              item-height="45"
            >
              <template v-slot:default="{ item }">
                <v-list-item :key="item.Id" class="customListItem">
                  <v-chip
                    color="white"
                    @click="
                      transport(ownedCategories, notOwnedCategories, item)
                    "
                  >
                    {{ item.Name }}
                  </v-chip>
                </v-list-item>
              </template>
            </v-virtual-scroll>
          </v-container>
          <v-container v-else>No attached categories</v-container>
        </v-card>

        <v-card class="pa-2 mx-2">
          <v-card-text> Available Tags</v-card-text>
          <v-divider />
          <!-- list of the tags if any exist, otherwise relevant message -->
          <v-container v-if="filteredTags && filteredTags.length > 0">
            <v-virtual-scroll
              :bench="1"
              :items="filteredTags"
              :height="listedItemsHeight"
              width="200"
              item-height="45"
            >
              <template v-slot:default="{ item }">
                <v-list-item :key="item.Id" class="customListItem">
                  <v-chip
                    color="white"
                    @click="transport(notOwnedTags, ownedTags, item)"
                  >
                    {{ item.Name }}
                  </v-chip>
                </v-list-item>
              </template>
            </v-virtual-scroll>
          </v-container>
          <v-container v-else>No availabe tags</v-container>
        </v-card>

        <v-card class="pa-2 mx-2">
          <v-card-text> Available Categories</v-card-text>
          <v-divider />
          <!-- list of the categories if any exist, otherwise relevant message -->
          <v-container
            v-if="filteredCategories && filteredCategories.length > 0"
          >
            <v-virtual-scroll
              :bench="1"
              :items="filteredCategories"
              :height="listedItemsHeight"
              width="200"
              item-height="45"
            >
              <template v-slot:default="{ item }">
                <v-list-item :key="item.Id" class="customListItem">
                  <v-chip
                    color="white"
                    @click="
                      transport(notOwnedCategories, ownedCategories, item)
                    "
                  >
                    {{ item.Name }}
                  </v-chip>
                </v-list-item>
              </template>
            </v-virtual-scroll>
          </v-container>
          <v-container v-else>No availabe categories</v-container>
        </v-card>
      </v-row>

      <v-row>
        <v-col class="d-flex justify-end">
          <v-btn
            class="mr-5"
            depressed
            color="grey lighten-2"
            @click="fileDetails ? forceClose() : closeWindow()"
          >
            {{ fileDetails ? "Ok" : "Cancel" }}</v-btn
          >
          <v-btn
            v-if="!fileDetails"
            depressed
            color="success"
            @click="SubmitAndExit"
          >
            Submit
          </v-btn>
        </v-col>
      </v-row>

      <v-dialog v-model="newTag">
        <add-dialog
          @closeAddDialog="(e) => (newTag = e)"
          @addItemFromDialog="(itemName) => addNewTagFromDialog(itemName)"
          :icons="icons"
          :ownedArray="ownedTags"
          :notOwnedArray="notOwnedTags"
          :nameOfInstance="'tag'"
        ></add-dialog>
      </v-dialog>

      <v-dialog v-model="newCategory">
        <add-dialog
          @closeAddDialog="(e) => (newCategory = e)"
          @addItemFromDialog="(itemName) => addNewCategoryFromDialog(itemName)"
          :icons="icons"
          :ownedArray="ownedCategories"
          :notOwnedArray="notOwnedCategories"
          :nameOfInstance="'category'"
        ></add-dialog>
      </v-dialog>

      <v-dialog
        v-if="!fileDetails"
        v-model="confirmExit"
        persistent
        no-click-animation
      >
        <confirm-exit
          @returnBack="(e) => (confirmExit = e)"
          @confirmedExit="(e) => forceCloseFromConfirmExit()"
          :icons="icons"
          :file="file"
          :ownedTags="ownedTags"
          :ownedCategories="ownedCategories"
          :path="path"
          :endpoints="endpoints"
          :axios="axios"
        ></confirm-exit>
      </v-dialog>
    </v-card>
  </v-overlay>
</template>

<script>
import AddDialog from "./tagsAndCategoriesComponents/AddDialog.vue";
import ConfirmExit from "./tagsAndCategoriesComponents/ConfirmExit.vue";

export default {
  props: {
    icons: Object,
    storage: String,
    path: String,
    endpoints: Object,
    axios: Function,
    file: Object,
    ownedTags: Array,
    notOwnedTags: Array,
    ownedCategories: Array,
    notOwnedCategories: Array,
    fileDetails: Boolean,
    startingTags: Array,
    startingCategories: Array,
  },
  components: {
    AddDialog,
    ConfirmExit,
  },
  emits: ["closeDialogTagsAndCategories"],
  data() {
    return {
      loading: false,
      items: [],
      select: [],
      URL: "",
      newTag: false,
      newCategory: false,
      guidCounter: 0,
      categoriesFilter: "",
      availableFilter: "",
      confirmExit: false,
    };
  },
  computed: {
    filteredTags() {
      return this.availableFilter == ""
        ? this.notOwnedTags
        : this.notOwnedTags.filter((x) =>
            x.Name.toLowerCase().includes(this.availableFilter.toLowerCase())
          );
    },
    filteredCategories() {
      return this.availableFilter == ""
        ? this.notOwnedCategories
        : this.notOwnedCategories.filter((x) =>
            x.Name.toLowerCase().includes(this.availableFilter.toLowerCase())
          );
    },
    listedItemsHeight: function () {
      switch (this.$vuetify.breakpoint.name) {
        case "sm":
          return 100;
        case "md":
          return 200;
        case "lg":
          return 300;
        case "xl":
          return 400;
        default:
          return 50;
      }
    },
  },
  watch: {
    select(val, prev) {
      if (prev !== null && val.length === prev.length) {
        return;
      }
      this.select = val.map((v) => {
        if (typeof v === "string") {
          v = {
            text: v,
            value: v,
            disabled: false,
            divider: false,
            header: "",
          };
        }
        return v;
      });
    },
  },
  methods: {
    initialize() {
      for (const tag of this.ownedTags) {
        this.startingTags.push(tag);
      }
      for (const category of this.ownedCategories) {
        this.startingCategories.push(category);
      }
    },
    closeWindow() {
      if (
        this.compareArrays(this.startingTags, this.ownedTags) != 0 ||
        this.compareArrays(this.startingCategories, this.ownedCategories) != 0
      ) {
        this.confirmExit = true;
      } else {
        this.forceClose();
      }
    },
    forceCloseFromConfirmExit() {
      this.confirmExit = false;
      this.forceClose();
    },
    forceClose() {
      this.clear();
      this.$emit("closeDialogTagsAndCategories", false);
    },
    SubmitAndExit() {
      this.axios({
        method: "post",
        url: this.endpoints.postCategoriesAndTags.url,
        data: {
          FileId: this.file.Id,
          Tags: this.ownedTags,
          Categories: this.ownedCategories,
        },
      }).then(() => {
        this.forceClose();
      });
    },
    clear() {
      (this.loading = false),
        (this.items = []),
        (this.select = null),
        (this.URL = ""),
        (this.availableFilter = ""),
        (this.guidCounter = 0),
        (this.confirmExit = false);
    },
    transport(fromList, toList, item) {
      for (let i = 0; i < fromList.length; i++) {
        if (fromList[i] === item) {
          this.sortedArrayInsertion(toList, fromList[i]);
          fromList.splice(i, 1);
          break;
        }
      }
    },

    sortedArrayInsertion(array, element) {
      let i;
      for (i = 0; i < array.length; i++) {
        if (array[i].Name.localeCompare(element.Name) > 0) {
          break;
        }
      }
      if (i < array.length) {
        array.splice(i, 0, element);
      } else {
        array.push(element);
      }
    },
    openAddCategory() {
      this.newCategory = true;
    },
    openAddTag() {
      this.newTag = true;
    },
    addNewTagFromDialog(itemName) {
      this.sortedArrayInsertion(this.ownedTags, {
        Id: this.createPseudoGUID(),
        Name: itemName,
      });
    },
    addNewCategoryFromDialog(itemName) {
      this.sortedArrayInsertion(this.ownedCategories, {
        Id: this.createPseudoGUID(),
        Name: itemName,
      });
    },
    createPseudoGUID() {
      this.guidCounter++;
      let id = this.guidCounter.toString();
      while (id.length < 32) {
        id = "0" + id;
      }
      let temp = [
        id.slice(0, 8),
        "-",
        id.slice(8, 12),
        "-",
        id.slice(12, 16),
        "-",
        id.slice(16, 20),
        "-",
        id.slice(20),
      ].join("");
      return temp;
    },
    compareArrays(array1, array2) {
      if (array1.length != array2.length) {
        return -1;
      }
      for (let i = 0; i < array1.length; i++) {
        if (array1[i] !== array2[i]) {
          return -1;
        }
      }
      return 0;
    },
  },
};
</script>

<style scoped>
.customListParent {
  overflow-x: hidden;
  overflow-y: scroll;
}
</style>