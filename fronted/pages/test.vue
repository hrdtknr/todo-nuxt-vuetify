<template>
  <v-data-table
    :headers="headers"
    :items="todoList"
    sort-by="calories"
    class="elevation-1"
  >
    <template v-slot:top>
      <v-toolbar flat color="white">
        <v-toolbar-title>My CRUD</v-toolbar-title>
        <v-divider class="mx-4" inset vertical></v-divider>
        <v-spacer></v-spacer>
        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on, attrs }">
            <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on"
              >New Item</v-btn
            >
          </template>
          <v-card>
            <v-card-title>
              <span class="headline">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field
                      v-model="editedItem.name"
                      label="NAME"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="6" md="4">
                    <v-text-field
                      v-model="editedItem.todo"
                      label="TODO"
                    ></v-text-field>
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="close">Cancel</v-btn>
              <v-btn color="blue darken-1" text @click="save">Save</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:item.actions="{ item }">
      <v-icon small class="mr-2" @click="editItem(item)"> mdi-pencil </v-icon>
      <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
    </template>
    <template v-slot:no-data>
      <v-btn color="primary" @click="initialize">Reset</v-btn>
    </template>
  </v-data-table>
</template>

<script>
import axios from "axios";
export default {
  data: () => ({
    dialog: false,
    headers: [
      {
        text: "ID",
        align: "start",
        sortable: false,
        value: "id",
      },
      { text: "NAME", value: "name" },
      { text: "TODO", value: "todo" },
      { text: "Actions", value: "actions", sortable: false },
    ],
    todoList: [],
    editedIndex: -1,
    editedItem: {
      name: "",
      todo: "",
    },
    defaultItem: {
      name: "",
      todo: "",
    },
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "New Item" : "Edit Item";
    },
  },

  watch: {
    dialog(val) {
      val || this.close();
    },
  },

  created() {
    this.initialize();
  },

  methods: {
    initialize() {
      axios
        .get("http://localhost:8000/todoList")
        .then((response) => (this.todoList = response.data))
        .catch((error) => console.log(error));
    },

    editItem(item) {
      //ペンボタンを押した瞬間のフォームデータが送信されている（この書き方だと
      //item に入ってるのは更新前の情報
      this.editedIndex = this.todoList.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
      //console.log("item", item);
      //console.log("item.id", item.id);
      //console.log("item.name", item.name);
      //console.log("item.todo", item.todo);

      //console.log("editItem", this.editedItem);

      //昨日のはthisがないから宣言されてないって怒られたのでは
      //this.updateTodo(this.editedItem);
    },

    deleteItem(item) {
      const index = this.todoList.indexOf(item);
      //console.log("index", index);
      //console.log("item id", item.id);

      if (confirm("Are you sure you want to delete this item?")) {
        this.todoList.splice(index, 1);
        const params = { id: item.id };
        const qs = new URLSearchParams(params);
        axios
          .delete(`http://localhost:8000/todoList?${qs}`)
          .catch((error) => console.log(error));
      }
    },

    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    save() {
      //saveボタンを押した後にupdateとinsert処理だね
      //console.log("save this.editedItem", this.editedItem);
      //console.log("save this.editedItem.name", this.editedItem.name);
      //console.log("save this.editedItem.todo", this.editedItem.todo);
      //console.log("save this.editedIndex", this.editedIndex);
      if (this.editedIndex > -1) {
        console.log("if update"); //update の処理
        Object.assign(this.todoList[this.editedIndex], this.editedItem);
        axios
          .put("http://localhost:8000/todoList", {
            id: this.editedItem.id,
            name: this.editedItem.name,
            todo: this.editedItem.todo,
          })
          .catch((error) => console.log(error));
      } else {
        console.log("else insert"); //insert の処理
        //this.todoList.push(this.editedItem);
        axios
          .post("http://localhost:8000/todoList", {
            name: this.editedItem.name,
            todo: this.editedItem.name,
          })
          .then(() => this.initialize()) //id表示するのに必要
          .catch((error) => console.log(error));
      }
      this.close();
      this.test();
    },

    test() {
      console.log("test");
    },
  },
};
</script>
