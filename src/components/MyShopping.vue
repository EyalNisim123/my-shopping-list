<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="6">
        <v-card class="elevation-0">
        <div style="font-size: 1.5em; font-weight: bold;">Sopping List</div>
          <v-data-table
              :headers="headers"
              :items="ShoopList"
              :mobile-breakpoint="100"
              :hide-default-footer="true"
              no-data-text="No Fuond Items"
              no-results-text="No Fuond Items"
              :disable-pagination="true"
              item-key="GroseryItemId"
              class="elevation-0"
              >
              <template v-slot:[`item.GroseryItemId`]="props">
                <v-edit-dialog
                  :return-value.sync="props.item.GroseryItemId"
                  @open="open(props.item)"
                >
                  {{ ShoopList.indexOf(props.item) + 1 }}
                </v-edit-dialog>
              </template>
              <template v-slot:[`item.GroseryItemName`]="props">
                <v-edit-dialog
                  :return-value.sync="props.item.GroseryItemName"
                  @open="open(props.item)"
                >
                 <span v-bind:class="{ 'text-decor': props.item.Approve }">{{ props.item.GroseryItemName }}</span>
                </v-edit-dialog>
              </template>
              <template v-slot:[`item.GroseryItemPrice`]="props">
                <v-edit-dialog
                  :return-value.sync="props.item.GroseryItemPrice"
                  @open="open(props.item)"
                >
                <span class="span" v-bind:class="{ 'text-decor': props.item.Approve }">{{ props.item.GroseryItemPrice + " " + props.item.GroseryPriceDesc }}</span>
                </v-edit-dialog>
              </template>
              <template v-slot:[`item.actions`]="props">
                <v-edit-dialog>
                  <v-icon
                    small
                    class="mr-2"
                    @click="editItem(props.item)"
                  >
                    mdi-pencil mdi-24px
                  </v-icon> |
                  <v-icon
                    small
                    @click="deleteItem(props.item)"
                  >
                    mdi-delete mdi-24px
                  </v-icon>
                 </v-edit-dialog>
              </template>
              <template v-slot:[`body.append`]="props" v-bind="sumOfProduct(props.items[0].GroseryItemPrice)">
                <span class="total">Total: {{ TotalPrice }}</span>
              </template>
          </v-data-table>
            <v-btn
              dark
              icon
              color="orange"
              class="btn-add"
              @click="openAddDialog"
            >
            <v-icon>mdi-plus</v-icon>
              Add Product
            </v-btn>
        </v-card>
      
      <v-dialog v-model="dialog" max-width="500px">
        <v-card>
          <v-card-title>
            <span class="text-h5">{{ formTitle }}</span>
          </v-card-title>

          <v-card-text>
            <v-container>
              <v-row>
                <v-col
                  cols="12"
                  sm="6"
                >
                  <v-text-field
                    v-model="editedItem.GroseryItemName"
                    label="Grosery Name"
                  ></v-text-field>
                </v-col>
                <v-col
                  cols="12"
                  sm="6"
                >
                  <v-text-field
                    v-model="editedItem.GroseryItemPrice"
                    label="Grosery Price"
                    @keypress="onlyNumber"
                  ></v-text-field>
                </v-col>
              </v-row>
              <v-row>
                <v-col
                  cols="12"
                >
                  <v-text-field
                    v-model="editedItem.Description"
                    label="Description"
                  ></v-text-field>
                </v-col>

              </v-row>
            </v-container>
          </v-card-text>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
              color="blue darken-1"
              text
              @click="close"
            >
              Cancel
            </v-btn>
            <v-btn
              color="blue darken-1"
              text
              @click="save"
            >
              Save
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <v-dialog v-model="dialogDelete" max-width="500px">
        <v-card>
          <v-card-title class="text-center text-h5">Are you sure you want to delete this item?</v-card-title>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" text @click="closeDelete">Cancel</v-btn>
            <v-btn color="blue darken-1" text @click="deleteItemConfirm">OK</v-btn>
            <v-spacer></v-spacer>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <v-dialog v-model="dialogDescription" max-width="500px">
        <v-card>
          <v-card-title class="text-center text-h5">Item Details</v-card-title>
          <v-card-text class="text-center text-h5">{{editedItem.Description}}</v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" text @click="closeDescription">Cancel</v-btn>
            <v-spacer></v-spacer>
          </v-card-actions>
        </v-card>
      </v-dialog>
   </v-col>
    </v-row>
  </v-container>
</template>

<script>
  export default {
    name: 'HelloWorld',
    data: () => ({
      ShoopList: [],
      headers:[
        {text: '', value: 'GroseryItemId', sortable: false, cellClass: "start-cell-class" },
        {text: '', value: 'GroseryItemName', sortable: false, cellClass: "cell-class", align: 'end' },
        {text: '', value: 'GroseryItemPrice', sortable: false, cellClass: "cell-class" },
        {text: '', value: 'GroseryPriceDesc', fixed: true, sortable: false, align: ' d-none', width: '10px' },
        {text: '', value: 'Description', fixed: true, sortable: false, align: ' d-none', width: '10px'},
        {text: '', value: 'Approve', fixed: true, sortable: false, align: ' d-none', width: '10px'},
        {text: '', value: 'actions', sortable: false },
      ],
      editedItem: {},
      editedIndex: -1,
      TotalPrice: 0,
      dialog: false,
      dialogDelete: false,
      dialogDescription: false
    }),
    computed: {
      formTitle () {
        return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
      },
    },
    created () {
      this.initialize();
    },
    methods:{
      openAddDialog(){
         this.$nextTick(() => {
          this.editedItem = {};
          this.editedIndex = -1
        });
        this.dialog = true;
      },
      editItem (item) {
        this.editedIndex = this.ShoopList.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.editedItem.Approve = true;
        Object.assign(this.ShoopList[this.editedIndex], this.editedItem)
      },
      deleteItemConfirm () {
        this.ShoopList.splice(this.editedIndex, 1);
        this.sumOfProduct();
        this.closeDelete()
      },
      deleteItem (item) {
        this.editedIndex = this.ShoopList.indexOf(item)
        this.editedItem = Object.assign({}, item);
        this.dialogDelete = true
      },
      initialize (){
        this.ShoopList = [
          {
            GroseryItemId: 1,
            GroseryItemName: 'Tomatos',
            GroseryItemPrice: 5,
            GroseryPriceDesc: 'NIS',
            Description: 'Hey Im Tomatos Desc',
            Approve: true
          },
          {
            GroseryItemId: 2,
            GroseryItemName: 'Cucombers',
            GroseryItemPrice: 3,
            GroseryPriceDesc: 'NIS',
            Description: 'Hey Im Cucombers Desc',
            Approve: false
          },
          {
            GroseryItemId: 3,
            GroseryItemName: 'Bread',
            GroseryItemPrice: 10,
            GroseryPriceDesc: 'NIS',
            Description: 'Hey Im Bread Desc',
            Approve: false
          },
          {
            GroseryItemId: 4,
            GroseryItemName: 'Grapes',
            GroseryItemPrice: 4,
            GroseryPriceDesc: 'NIS',
            Description: 'Hey Im Grapes Desc',
            Approve: false
          }
        ];
        this.sumOfProduct()
      },
      close () {
        this.dialog = false
        this.$nextTick(() => {
          this.editedItem = {};
          this.editedIndex = -1
        })
      },
      closeDelete () {
        this.dialogDelete = false
        this.$nextTick(() => {
          this.editedItem = {};
          this.editedIndex = -1
        })
      },
      closeDescription(){
        this.dialogDescription = false;
        this.$nextTick(() => {
          this.editedItem = {};
          this.editedIndex = -1
        });
      },
      save () {
        if (this.editedIndex > -1) {
           this.editedItem.Approve = false;
          Object.assign(this.ShoopList[this.editedIndex], this.editedItem)
        } else {
          this.editedItem.Approve = false;
          this.editedItem.GroseryItemId = this.ShoopList.length;
          this.editedItem.GroseryPriceDesc = 'NIS'
          this.ShoopList.push(this.editedItem)
        }
        this.sumOfProduct();
        this.close()
      },
      open(item){
        if(item != null){
          if(item.Description != ''){
            this.editedIndex = this.ShoopList.indexOf(item)
            this.editedItem = Object.assign({}, item)
            this.dialogDescription = true;
          }else{
            alert('No Description Exist')
          }
        }
      },
      onlyNumber(event) {
        let keyCode = event.keyCode;
        if (keyCode >= 48 && keyCode <= 57 || keyCode == 44) {
            return;
        }
        event.preventDefault();
      },
      sumOfProduct(item){
        var x = 0;
        x = item;
        item = x;
        this.TotalPrice = 0;
          if (this.ShoopList.length > 0) {
              for (var i = 0; i < this.ShoopList.length; i++) {
                  this.TotalPrice = parseFloat(this.TotalPrice) + parseFloat(this.ShoopList[i].GroseryItemPrice)
              }
          } else {
              this.TotalPrice = 0;
          }
        return  this.TotalPrice
      }
    }
  }
</script>

<style lang="css">
 @import "../style/style.css"
</style>
