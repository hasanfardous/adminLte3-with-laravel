<template>
    <div class="container">
        <div class="row mt-5" v-if="$gate.isAdminOrAuthor()">
          <div class="col-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">Users Table</h3>

                <div class="card-tools">
                  <button class="btn btn-success" @click="newModal">
                      Add New <i class="fa fa-user-plus fa-fw"></i>
                  </button>
                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover">
                  <tbody><tr>
                    <th>ID</th>
                    <th>Name</th>
                    <th>Email</th>
                    <th>Type</th>
                    <th>Registered At</th>
                    <th>Action</th>
                  </tr>

                  <tr v-for="user in users.data" :key="user.id">
                    <td>{{ user.id }}</td>
                    <td>{{ user.name }}</td>
                    <td>{{ user.email }}</td>
                    <td>{{ user.type | upText }}</td>
                    <td>{{ user.created_at | myDate }}</td>
                    <td>
                        <a href="#" @click="editModal(user)"><i class="fa fa-edit text-blue"></i></a> | 
                        <a href="#" @click="deleteUser(user.id)"><i class="fa fa-trash text-red"></i></a>
                    </td>
                  </tr>
                </tbody></table>
              </div>
              <!-- /.card-body -->

              <div class="card-footer">
                  <pagination :data="users" @pagination-change-page="getResults"></pagination>
              </div>
            </div>
            <!-- /.card -->
          </div>
        </div>

        <div v-if="!$gate.isAdminOrAuthor()">
            <not-found></not-found>
        </div>

        <form @submit.prevent="editMode ? updateUser() : createUser()">
        <!-- Modal -->
        <div class="modal fade" id="adminModal" tabindex="-1" role="dialog" aria-labelledby="adminModalLabel" aria-hidden="true">
        <div class="modal-dialog modal-dialog-centered" role="document">
            <div class="modal-content">
            <div class="modal-header">
                <h5 class="modal-title" v-show="!editMode" id="adminModalLabel">Add New User</h5>
                <h5 class="modal-title" v-show="editMode" id="adminModalLabel">Update the User</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                <span aria-hidden="true">&times;</span>
                </button>
            </div>
            <div class="modal-body">
                <div class="form-group">
                    <input v-model="form.name" type="text" name="name" placeholder="Name"
                        class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                    <has-error :form="form" field="name"></has-error>
                </div>
                <div class="form-group">
                    <input v-model="form.email" type="email" name="email" placeholder="Email address"
                        class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                    <has-error :form="form" field="email"></has-error>
                </div>
                <div class="form-group">
                    <input v-model="form.password" type="password" name="password" placeholder="Password"
                        class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                    <has-error :form="form" field="password"></has-error>
                </div>
                <div class="form-group">
                    <select v-model="form.type" id="type" name="type"
                        class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                        <option value="">Select User Type</option>
                        <option value="admin">Admin</option>
                        <option value="user">Standard user</option>
                        <option value="writer">Writer</option>
                    </select>
                    <has-error :form="form" field="type"></has-error>
                </div>
                <div class="form-group">
                    <textarea v-model="form.bio" type="bio" name="bio" placeholder="Write something about you. (Optional)"
                        class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }">
                    </textarea>
                    <has-error :form="form" field="bio"></has-error>
                </div>
                <!-- <div class="form-group">
                    <input v-model="form.photo" type="photo" name="photo" placeholder="photo address"
                        class="form-control" :class="{ 'is-invalid': form.errors.has('photo') }">
                    <has-error :form="form" field="photo"></has-error>
                </div> -->
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-secondary btn-danger" data-dismiss="modal">Close</button>
                <button type="submit" v-show="!editMode" class="btn btn-primary">Save</button>
                <button type="submit" v-show="editMode" class="btn btn-success">Update</button>
            </div>
            </div>
        </div>
        </div>
        </form>
    </div>
</template>

<script>
    import Form from 'vform'; 
    
    export default {
        data () {
            return {
                editMode: false,
                users: {},
                form: new Form({
                    id: '',
                    name: '',
                    email: '',
                    password: '',
                    type: '',
                    bio: '',
                    photo: '',
                })
            }
        },

        methods: {
            getResults(page = 1){
                axios.get('api/user?page=' + page)
				.then(response => {
					this.users = response.data;
				});
            },
            newModal(){
                this.editMode = false;
                this.form.reset();
                $('#adminModal').modal('show');
            },

            editModal(user){
                this.editMode = true;
                this.form.reset();
                $('#adminModal').modal('show');
                this.form.fill(user);
            },
            
            updateUser(){
                this.$Progress.start();
                this.form.put('api/user/'+this.form.id)
                .then(() => {
                    Fire.$emit('AfterCreated');
                    $('#adminModal').modal('hide');

                    Toast.fire({
                        type: 'success',
                        title: 'User updated in Successfully'
                    });

                    this.$Progress.finish();
                })
                .catch(() => {
                    // Toast.fire({
                    //     type: 'error',
                    //     title: 'Sorry! Error happend, try again.'
                    // });
                    this.$Progress.fail();
                })
            },
            
            createUser(){
                // this.editMode = false;
                this.$Progress.start();
                this.form.post('api/user')
                .then(() => {
                    Fire.$emit('AfterCreated');
                    $('#adminModal').modal('hide');

                    Toast.fire({
                        type: 'success',
                        title: 'User created in Successfully'
                    });

                    this.$Progress.finish();
                })
                .catch(() => {
                    Toast.fire({
                        type: 'error',
                        title: 'Sorry! Error happend, try again.'
                    });
                })

            },

            deleteUser(id){
                Swal.fire({
                    title: 'Are you sure?',
                    text: "You won't be able to delete this!",
                    type: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Yes, delete it!'
                }).then((result) => {
                    if (result.value) {
                        this.form.delete('api/user/'+id).then(()=> {
                            Swal.fire(
                            'Deleted!',
                            'User has been deleted.',
                            'success'
                            )
                            Fire.$emit('AfterCreated');
                        }).catch(() => {
                            Swal.fire(
                                'Failed!',
                                'There was something wrong.',
                                'warning'
                            )
                        })
                    }

                })
            },
            
            loadUsers() {
                if (this.$gate.isAdminOrAuthor()) {
                    axios.get('api/user').then(({data}) => (this.users = data));
                }
            },
        },
        
        created() {
            Fire.$on('searching', () => {
                let query = this.$parent.search;
                axios.get('api/findUser?q=' + query)
                .then((data) => {
                    this.users = data.data
                })
                .catch(() => {

                })
            })
            this.loadUsers();
            // setInterval(() => this.loadUsers(), 3000);
            Fire.$on('AfterCreated', () => {
                this.loadUsers();
            });
        }
    }
</script>
