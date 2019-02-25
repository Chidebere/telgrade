<template>
    <div class="container">
        <div class="row justify-content-center mt-5">

            <div class="col-md-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">User List</h3>

                <div class="card-tools">
                    <button class="btn btn-success" @click="newModel"> <i class="fa fa-user-plus"></i> Add New Users</button>
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
                    <th>Created on</th>
                    <th>Modify</th>
                  </tr>
                  <tr v-for="user in users" :key="user.id">
                    <td>{{ user.id }}</td>
                    <td>{{ user.name }}</td>
                    <td>{{ user.email }}</td>
                    <td>{{ user.type | upText }}</td>
                    <td>{{ user.created_at | myDate }}</td>


                    <td>
                     <a href="#" @click="editModel(user)">
                         <i class="fa fa-edit blue"></i>
                     </a>
                        |
                    <a href="#" @click="deleteUser(user.id)">
                        <i class="fa fa-trash red"></i>
                    </a>

                    </td>
                  </tr>

                </tbody></table>
              </div>
              <!-- /.card-body -->
            </div>
            <!-- /.card -->
          </div>

        </div>



<!-- Modal -->
<div class="modal fade" id="exampleModal" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
  <div class="modal-dialog modal-dialog-centered" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title" v-show="!editMode" id="exampleModalLabel">Add New User</h5>
        <h5 class="modal-title" v-show="editMode" id="exampleModalLabel">Update User Record</h5>
        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      <div class="modal-body">


       <form @submit.prevent="editMode ? updateUser() : createUser()">

        <div class="form-group">

            <input v-model="form.name" type="text" name="name"
                class="form-control" placeholder="Name" :class="{ 'is-invalid': form.errors.has('name') }">
            <has-error :form="form" field="name"></has-error>
        </div>

        <div class="form-group">
            <input v-model="form.email" type="email" name="email"
                class="form-control" placeholder="Email" :class="{ 'is-invalid': form.errors.has('email') }">
            <has-error :form="form" field="email"></has-error>
        </div>

        <div class="form-group">
            <input v-model="form.password" type="password" name="password"
                class="form-control" placeholder="Password" :class="{ 'is-invalid': form.errors.has('password') }">
            <has-error :form="form" field="password"></has-error>
        </div>

        <div class="form-group">
           <textarea  v-model="form.bio" name="bio"
                class="form-control" placeholder="Short Bio for the User (optional)" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
            <has-error :form="form" field="bio"></has-error>
        </div>

        <div class="form-group">
            <select v-model="form.type" name="type"
            class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
            <option value="">Select User Role</option>
            <option value="admin">Admin</option>
            <option value="author">Author</option>
            <option value="user">Standard User</option>
            </select>
            <has-error :form="form" field="type"></has-error>
        </div>

            <div class="modal-footer">
                <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                <button type="submit" v-show="!editMode" class="btn btn-primary">Save</button>
                <button type="submit" v-show="editMode" class="btn btn-success">Update</button>
            </div>

        </form>

      </div>

    </div>
  </div>
</div>
<!-- End Modal -->

    </div>
</template>

<script>
export default {
    data () {
        return {
            editMode:false,
            users: {},
        // Create a new form instance
            form: new Form({
                    id: '',
                    name: '',
                    email: '',
                    password: '',
                    type: '',
                    bio: '',
                   // photo: ''
                    })
                    }
                },
            methods: {
                updateUser(){
                    this.form.put('api/user/' + this.form.id)
                    .then(()=>{
                        $('#exampleModal').modal('hide');
                        this.$Progress.start();

                        toast.fire({
                        type: 'success',
                        title: 'User update was successful'
                        })

                        Fire.$emit('EventAction');

                    })
                    .catch(()=>{
                        this.$Progress.fail();
                    })
                },
                editModel(user){
                    this.editMode = true;
                    this.form.reset();
                   $('#exampleModal').modal('show'); //open model
                   this.form.fill(user);
                },
                newModel(){
                    this.editMode = false
                    this.form.reset();
                   $('#exampleModal').modal('show'); //open model
                },
                //Delete a user
                deleteUser(id){
                    Swal.fire({
                        title: 'Are you sure?',
                        text: "You won't be able to revert this!",
                        type: 'warning',
                        showCancelButton: true,
                        confirmButtonColor: '#3085d6',
                        cancelButtonColor: '#d33',
                        confirmButtonText: 'Yes, delete it!'
                        }).then((result) => {
                            //send the request to the server
                        if (result.value) {
                            this.form.delete('api/user/' + id)
                            .then(() => {

                                    Swal.fire(
                                    'Deleted!',
                                    'Your file has been deleted.',
                                    'success'
                                    )
                                Fire.$emit('EventAction');
                            })

                            .catch(() => {
                                Swal("Failed!", "There was something wrong", "warning");
                            })
                        }
                    })
                },
                loadUsers() {
                    this.$Progress.start();
                    axios.get('api/user')
                    .then(({ data }) => (this.users = data.data));
                    this.$Progress.finish();
                },
                createUser() {
                // Submit the form via a POST request
                this.$Progress.start(); //start progress bar
                this.form.post('api/user')
                .then(()=>{
                     Fire.$emit('EventAction');
                    $('#exampleModal').modal('hide'); //Hide model after submit

                    // Display SweetAlert message dialog
                    toast.fire({
                    type: 'success',
                    title: 'User created successfully'
                    })
                    this.$Progress.finish(); // end progress bar
                })
                .catch(()=>{

                })

                    }
                },
                created() {
                        this.loadUsers();
                        Fire.$on('EventAction', () => {
                            this.loadUsers();
                        }); //Listening to the custom event triggered when a user is created

                        //setInterval(() => this.loadUsers(), 3000); //constant firing event after 3 seconds

                        }
                    }
</script>
