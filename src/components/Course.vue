<template>
    <div>
      <v-data-table
        :headers="headers"
        :items="courseItem"
        class="elevation-1"
      >
        <template v-slot:top>
          <v-toolbar flat>
            <v-toolbar-title>Table Course</v-toolbar-title>
            <v-divider class="mx-4" inset vertical></v-divider>
            <v-spacer></v-spacer>
            <v-btn color="primary" dark class="mb-2" @click="openDialog('add', defaultItem)">
              เพิ่มข้อมูล
            </v-btn>
          </v-toolbar>
        </template>
        <template v-slot:[`item.actions`] = "{ item }">
          <v-btn small outlined @click="openDialog('edit', item)" color="blue">
            <v-icon>mdi-pencil</v-icon>
          </v-btn>
          <v-btn small outlined @click="deleteItem(item)" color="red" class="ml-2">
            <v-icon>mdi-delete</v-icon>
          </v-btn>
        </template>
        <template v-slot:no-data>
          <v-btn color="primary" @click="initialize">Reset</v-btn>
        </template>
      </v-data-table>
      <v-dialog v-model="dialogCreate" max-width="500px">
        <v-card>
          <v-card-title>
            <span class="text-h5">{{ formTitle }}</span>
          </v-card-title>
          <v-card-text>
            <v-container>
              <v-row>
                <v-col cols="12" sm="6" md="6">
                  <v-text-field v-model="editedItem.courseID" label="รหัสคอร์ส"></v-text-field>
                </v-col>
                <v-col cols="12" sm="6" md="6">
                  <v-text-field v-model="editedItem.courseName" label="ชื่อคอร์ส"></v-text-field>
                </v-col>
                <v-col cols="12" sm="6" md="6">
                  <v-text-field v-model="editedItem.courseDetail" label="รายละเอียดคอร์ส"></v-text-field>
                </v-col>
              </v-row>
            </v-container>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" text @click="close">ยกเลิก</v-btn>
            <v-btn color="blue darken-1" text @click="saveData">{{ formTitle }}</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      <v-dialog v-model="dialogDelete" max-width="500px">
        <v-card>
          <v-card-title class="text-h5">ตุณต้องการลบข้อมูลนี้ในตารางใช่ หรือ ไม่?</v-card-title>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" text @click="closeDelete">ยกเลิก</v-btn>
            <v-btn color="blue darken-1" text @click="deleteItemConfirm">ตกลง</v-btn>
            <v-spacer></v-spacer>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </div>
  </template>
  
  <script>
  export default {
    data: () => ({
      dialogCreate: false,
      dialogDelete: false,
      headers: [
        {
          text: 'ไอดี',
          align: 'start',
          sortable: false,
          value: 'id'
        },
        { text: 'รหัสคอร์ส', value: 'courseID' },
        { text: 'ชื่อคอร์ส', value: 'courseName' },
        { text: 'รายละเอียดคอร์ส', value: 'courseDetail' },
        { text: 'จัดการ', value: 'actions', sortable: false }
      ],
      courseItem: [],
      editedItem: {
        courseID: '',
        courseName: '',
        courseDetail: ''
      },
      defaultItem: {
        courseID: '',
        courseName: '',
        courseDetail: ''
      },
      formTitle: '',
      idforDelete: ''
    }),
  
    watch: {
      dialogCreate(val) {
        val || this.close();
      },
      dialogDelete(val) {
        val || this.closeDelete();
      }
    },
  
    created() {
      this.initialize();
    },
  
    methods: {
      async initialize() {
        this.courseItem = [];
        try {
          var data = await this.axios.get('http://localhost:9000/course');
          console.log('data course ====>', data);
          this.courseItem = data.data;
        } catch (error) {
          console.log(error.message);
        }
      },
  
      openDialog(Action, item) {
        this.formTitle = '';
        if (Action === 'add') {
          this.dialogCreate = true;
          this.formTitle = 'เพิ่มข้อมูล';
          this.editedItem = { ...this.defaultItem };
        } else {
          this.formTitle = 'แก้ไขข้อมูล';
          this.dialogCreate = true;
          this.editedItem = { ...item };
        }
      },
  
      deleteItem(item) {
        this.idforDelete = item.id;
        this.dialogDelete = true;
      },
  
      async deleteItemConfirm() {
        try {
          var response = await this.axios.delete('http://localhost:9000/course/' + this.idforDelete);
          this.initialize();
        } catch (error) {
          console.log(error.message);
        }
        this.closeDelete();
      },
  
      close() {
        this.dialogCreate = false;
        this.editedItem = { ...this.defaultItem };
      },
  
      closeDelete() {
        this.dialogDelete = false;
      },
  
      async saveData() {
        var data = {
          courseID: this.editedItem.courseID,
          courseName: this.editedItem.courseName,
          courseDetail: this.editedItem.courseDetail
        };
        try {
          if (this.formTitle === 'เพิ่มข้อมูล') {
            await this.axios.post('http://localhost:9000/course', data);
          } else {
            await this.axios.put('http://localhost:9000/course/' + this.editedItem.id, data);
          }
          this.close();
          this.initialize();
        } catch (error) {
          console.log(error.message);
        }
      }
    }
  };
  </script>
  
  <style>
  </style>
  