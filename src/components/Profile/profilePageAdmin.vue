<template>
    <v-main class="list">
        <h3 class="text-h3 font-weight-medium mb-5">Daftar User</h3>

        <v-card>
            <v-card-title>
                <v-text-field v-model="search" append-icon="mdi-magnify" label="search" single-line hide-details="">
                </v-text-field>
                <v-spacer></v-spacer>
            </v-card-title>
            <v-data-table :headers="headers" :items="users" :search="search">
                <template v-slot:[`item.actions`]="{item}">
                    <v-btn small class="mr-2" @click="editHandler(item)">edit</v-btn>
                    <v-btn small @click="deleteHandler(item.id)">delete</v-btn>
                </template>
            </v-data-table>
        </v-card>

        <v-dialog v-model="dialog" persistent width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline">{{ formTitle }} User</span>
                </v-card-title>
                <v-card-text>
                    <v-container>
                        <v-text-field v-model="form.namaLengkap" label="Nama Lengkap" required></v-text-field>
                        <v-text-field v-model="form.email" label="Email" required type="email"></v-text-field>
                        <v-text-field v-model="form.username" label="Username" required>
                        <v-text-field v-model="form.noTelp" label="Nomor Telepon" required></v-text-field>
                        </v-text-field>
                    </v-container>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="cancel">Cancel</v-btn>
                    <v-btn color="blue darken-1" text @click="setForm">Save</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <v-dialog v-model="dialogEdit" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline">{{ formTitle }} User</span>
                </v-card-title>
                <v-card-text>
                    <v-container>
                        <v-text-field v-model="form.namaLengkap" label="Nama Lengkap" required></v-text-field>
                        <v-text-field v-model="form.email" label="Email" required type="email"></v-text-field>
                        <v-text-field v-model="form.username" label="Username" required>
                        <v-text-field v-model="form.noTelp" label="Nomor Telepon" required></v-text-field>
                        </v-text-field>
                    </v-container>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="cancel">Cancel</v-btn>
                    <v-btn color="blue darken-1" text @click="setForm">Save</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <v-dialog v-model="dialogConfirm" persistent max-width="400px">
            <v-card>
                <v-card-title>
                    <span class="headline">Warning!</span>
                </v-card-title>
                <v-card-text>Anda yakin ingin menghapus profil ini?</v-card-text>
                <v-card-action>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="dialogConfirm= false">Cancel</v-btn>
                    <v-btn color="blue darken-1" text @click="deleteData">Delete</v-btn>
                </v-card-action>
            </v-card>
        </v-dialog>

        <v-snackbar v-model="snackbar" :color="color" timeout="2000" bottom>{{ error_message }}</v-snackbar>
    </v-main>
</template>

<script>
    export default {
        name: "ProfileAdmin",
        data() {
            return {
                inputType: 'Tambah',
                load: false,
                snackbar: false,
                error_message: '',
                color: '',
                search: null,
                dialog: false,
                dialogEdit: false,
                dialogConfirm: false,
                headers: [{
                        text: "Nama Lengkap",
                        align: "start",
                        sortable: true,
                        value: 'namaLengkap'
                    },
                    {
                        text: "Email",
                        value: 'email'
                    },
                    {
                        text: "Username",
                        value: 'username'
                    },
                    {
                        text: "Nomor Telepon",
                        value: 'noTelp'
                    },
                    {
                        text: "Actions",
                        value: 'actions'
                    },
                ],
                user: new FormData,
                users: [],
                form: {
                    name: null,
                    email: null,
                    password: null,
                },
                deleteId: '',
                editId: ''
            };
        },
        methods: {
            setForm() {
                if (this.inputType !== 'Tambah') {
                    this.update();
                } else {
                    this.save();
                }
            },
            //read data
            readData() {
                var url = this.$api + '/user';
                this.$http.get(url, {
                    headers: {
                        'Authorization': 'Bearer ' + localStorage.getItem('token')
                    }
                }).then(response => {
                    this.users = response.data.data;
                })
            },
            //simpan data
            save() {
                this.user.append('namaLengkap', this.form.namaLengkap);
                this.user.append('email', this.form.email);
                this.user.append('username', this.form.username);
                this.user.append('noTelp', this.form.noTelp);

                var url = this.$api + '/user'
                this.load = true;
                this.$http.post(url, this.user, {
                    headers: {
                        'Authorization': 'Bearer ' + localStorage.getItem('token')
                    }
                }).then(response => {
                    this.error_message = response.data.message;
                    this.color = "green";
                    this.snackbar = true;
                    this.load = true;
                    this.close();
                    this.readData();
                    this.resetForm();
                }).catch(error => {
                    this.error_message = error.response.data.message;
                    this.color = "red",
                        this.snackbar = true;
                    this.load = false
                });
            },
            //update data
            update() {
                let newData = {
                    namaLengkap: this.form.namaLengkap,
                    email: this.form.email,
                    username: this.form.username,
                    noTelp: this.form.noTelp
                };
                var url = this.$api + '/user/' + this.editId;
                this.load = true;
                this.$http.put(url, newData, {
                    headers: {
                        'Authorization': 'Bearer ' + localStorage.getItem('token')
                    }
                }).then(response => {
                    this.error_message = response.data.message;
                    this.color = "green";
                    this.snackbar = true;
                    this.load = false;
                    this.close();
                    this.readData();
                    this.resetForm();
                    this.inputType = 'Tambah';
                }).catch(error => {
                    this.error_message = error.response.data.message;
                    this.color = 'red';
                    this.snackbar = true;
                    this.load = false;
                });
            },
            //hapus data produk
            deleteData() {
                var url = this.$api + '/user/' + this.deleteId;
                this.load = true;
                this.$http.delete(url, {
                    headers: {
                        'Authorization': 'Bearer ' + localStorage.getItem('token')
                    }
                }).then(response => {
                    this.error_message = response.data.message;
                    this.color = "green";
                    this.snackbar = true;
                    this.load = false;
                    this.close();
                    this.readData();
                    this.resetForm();
                    this.inputType = "Tambah";
                }).catch(error => {
                    this.error_message = error.response.data.message;
                    this.color = "red";
                    this.snackbar = true;
                    this.load = false;
                });
            },
            editHandler(item) {
                this.inputType = "Ubah";
                this.editId = item.id;
                this.form.namaLengkap = item.namaLengkap;
                this.form.email = item.email;
                this.form.username = item.username;
                this.form.noTelp = item.noTelp;
                this.dialogEdit = true;
            },
            deleteHandler(id) {
                this.deleteId = id;
                this.dialogConfirm = true;
            },
            close() {
                this.dialog = false;
                this.dialogEdit = false;
                this.inputType = 'Tambah';
                this.dialogConfirm = false;
                this.readData();
            },
            cancel() {
                this.resetForm();
                this.readData();
                this.dialog = false;
                this.dialogEdit = false;
                this.dialogConfirm = false;
                this.inputType = 'Tambah';
            },
            resetForm() {
                this.form = {
                    name: null,
                    email: null,
                    password: null
                };
            },
        },
        computed: {
            formTitle() {
                return this.inputType;
            },
        },
        mounted() {
            this.readData();
        },
    };
</script>