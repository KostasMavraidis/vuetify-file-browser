<template >
  <v-overlay :absolute="true">
    <v-card flat light :loading="loading" class="pa-5">

      <v-row class="ma-0" > 
        <v-card-text class="text-h5 d-flex justify-center"> Share {{ file.name }} </v-card-text>
      </v-row>

      <v-row class="ma-0" >
        <v-toolbar flat height="86px">
          <v-toolbar-title>E-mail</v-toolbar-title>
          <v-combobox
            v-model="select"
            :loading="loading"
            :items="items"
            outlined
            :search-input.sync="search"
            cache-items
            class="mx-4"
            multiple
            hide-no-data
    
            label="Share User E-mail"
            solo-inverted
            chips
            deletable-chips
            clearable
          >
            <template v-slot:selection="{ attrs, item, parent, selected }">
              <v-chip
                v-if="item === Object(item)"
                v-bind="attrs"
                :input-value="selected"
                label
                small
              >
                <span class="pr-2">
                  {{ item.value }}
                </span>
                <v-icon small @click="parent.selectItem(item)">
                  $delete
                </v-icon>
              </v-chip>
            </template>
          </v-combobox>
        </v-toolbar>
      </v-row>

      <v-row class="ma-0" >
        <v-menu
          v-model="menu2"
          :close-on-content-click="false"
          :nudge-right="40"
          transition="scale-transition"
          offset-y
          min-width="auto"
        >
          <template v-slot:activator="{ on, attrs }">
            <v-text-field
              v-model="date"
              label="Expiration Date"
              prepend-icon="mdi-calendar"
              readonly
              v-bind="attrs"
              v-on="on"
            ></v-text-field>
          </template>
          <v-date-picker v-model="date" @input="menu2 = false"></v-date-picker>
        </v-menu>
      </v-row>

      <v-row class="ma-0">
        <v-text-field v-model="text1" ref="textToCopy"></v-text-field>
        <v-btn @click="copyText">copy</v-btn>
      </v-row>

      <v-row class="ma-0">
        <v-toolbar dense flat>
          <v-checkbox
            v-model="checkbox"
            :label="`Send E-mail to the Users`"
          ></v-checkbox>
        </v-toolbar>
      </v-row>

      <v-row class="ma-0">
        <v-toolbar dense flat color="white">
          <v-btn text @click="closeDialog" class="mx-1">
            <span class="grey--text">Cancel</span></v-btn
          >
          <v-btn depressed color="warning" @click="clear" class="mx-1">
            <v-icon>mdi-close</v-icon>Clear
          </v-btn>
          <v-btn depressed color="success" @click="Confirm" class="ml-1">
            Confirm
          </v-btn>
        </v-toolbar>
      </v-row>

      <v-toolbar dense flat color="white" v-if="appearMessage">
        <span class="red--text"
          >You are about to share a file with a guest if you want to proceed
          press again confirm</span
        >
      </v-toolbar>
      <v-toolbar dense flat color="white" v-if="ConfirmationMessageB">
        <span class="green--text">{{ ConfirmationMessage }}</span>
      </v-toolbar>
    </v-card>
  </v-overlay>
</template>

<script>
export default {
  props: {
    icons: Object,
    storage: String,
    path: String,
    endpoints: Object,
    axios: Function,
    file: Object,
  },
  emtis: ["close-dialog"],
  data() {
    return {
      loading: false,
      items: [],
      search: null,
      select:[],
      URL: "",
      currentValue: "",
      text1: "",
      date: new Date(Date.now() - new Date().getTimezoneOffset() * 60000)
        .toISOString()
        .substr(0, 10),
      menu: false,
      modal: false,
      menu2: false,
      appearMessage: false,
      checkbox: true,
      appearCheckbox: false,
      ConfirmationMessage: "",
      ConfirmationMessageB: false,
    };
  },
  watch: {
    select(val, prev) {
        if (prev!==null)
        {
     //console.log(val);
        ////console.log(prev);
      if (val.length === prev.length) return ;
        }
      this.select = val.map((v) => {
        if (typeof v === "string") {
            //console.log(v);
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
      //console.log(val);
      //console.log(this.select);
    },

    //   })

    search(val) {
      this.currentValue = val;
      this.items = [];
      //console.log(this.search);
      //console.log(this.path);
      val && val !== this.select && this.querySelections(val);
    },
  },
  methods: {
    async querySelections(v) {
      this.loading = true;
      // Simulated ajax query

      //console.log(this.endpoints);
      let url = this.endpoints.getEmail.url.replace(
        new RegExp("{Name}", "g"),
        v
      );
      let config = {
        url,
        method: this.endpoints.getEmail.method || "get",
      };

      //console.log(this.file);
      //console.log(this.items);
      let response = await this.axios.request(config);
      for (var i of response.data) {
        var obj = {
          text: "",
          value: "",
          disabled: false,
          divider: false,
          header: "",
        };

        obj.text = i.Name + " " + i.Email + " " + i.Type;
        obj.value = i.Email;

        this.items.push(obj);

        //this.items.text.push(o);
      }
      //console.log(this.items);

      this.loading = false;
    },
    closeDialog() {
      this.clear();
      this.$emit("closeDialog", false);
    },
    Confirm() {
      for (var y = 0; y < this.select.length; y++) {
        //console.log(this.select[y].value);
        if (
          this.select[y].value === this.select[y].text &&
          this.appearMessage === false
        ) {
          this.appearMessage = true;
          return;
        }
      }
      //console.log(this.select);

  
      var emails = [];
      for (var i = 0; i < this.select.length; i++) {
      
          emails.push(this.select[i].value);
        
      }

   
      this.dialog = false;
      this.appearMessage = false;
      this.ConfirmationMessageB = true;
      //console.log(this.ConfirmationMessage);
    },
    copyText() {
      let textToCopy = this.$refs.textToCopy.$el.querySelector("input");
      textToCopy.select();
      document.execCommand("copy");
    },
    async axiosRequest(a) {
      await this.axios({
        method: "post",
        url: this.endpoints.share.url,
        data: {
          ShouldBeSent: this.checkbox,
          contents: "",
          fileIds: [this.file.Id],
          receivers: a,
          expirationDate: `${this.date}`,
        },
      }).then(
        (response) => {
          //console.log(response);
          if (response.data.length !== 0) {
            this.text1 = response.data[0].FileName;
            //console.log(response.data[0].FileName);
          }
          if (response.statusText === "OK") {
            //console.log(response.statusText);
            this.ConfirmationMessage = "The file was shared succesfully";
          } else {
            //console.log(response.statusText);
            this.ConfirmationMessage = "Oops something went wrong";
          }
        },
        (error) => {
          console.log(error);
        }
      );
    },
    clear() {
      (this.loading = false),
        (this.items = []),
        (this.search = null),
        (this.select = null),
        (this.states = []),
        (this.URL = ""),
        (this.currentValue = ""),
        (this.text1 = ""),
        (this.date = new Date(
          Date.now() - new Date().getTimezoneOffset() * 60000
        )
          .toISOString()
          .substr(0, 10)),
        (this.menu = false),
        (this.modal = false),
        (this.menu2 = false),
        (this.appearMessage = false),
        (this.checkbox = true),
        (this.appearCheckbox = false),
        (this.ConfirmationMessage = ""),
        (this.ConfirmationMessageB = false);
    },
  },
};
</script>


