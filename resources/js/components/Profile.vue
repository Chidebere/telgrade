<template>
    <div class="container">
        <div class="row mt-5">
            <div class="col-md-12">



            <!-- Widget: user widget style 1 -->
        <div class="card card-widget widget-user">
              <!-- Add the bg color to the header using any of the bg-* classes -->
              <div class="widget-user-header text-white" style="background: url('./img/profile/user_cover.jpg') center center;">
                <h3 class="widget-user-username">Elizabeth Pierce</h3>
                <h5 class="widget-user-desc">Web Designer</h5>
              </div>
              <div class="widget-user-image">
                <img class="img-circle" :src="getProfilePhoto()" alt="User Avatar">
              </div>
              <div class="card-footer">
                <div class="row">
                  <div class="col-sm-4 border-right">
                    <div class="description-block">
                      <h5 class="description-header">3,200</h5>
                      <span class="description-text">SALES</span>
                    </div>
                    <!-- /.description-block -->
                  </div>
                  <!-- /.col -->
                  <div class="col-sm-4 border-right">
                    <div class="description-block">
                      <h5 class="description-header">13,000</h5>
                      <span class="description-text">FOLLOWERS</span>
                    </div>
                    <!-- /.description-block -->
                  </div>
                  <!-- /.col -->
                  <div class="col-sm-4">
                    <div class="description-block">
                      <h5 class="description-header">35</h5>
                      <span class="description-text">PRODUCTS</span>
                    </div>
                    <!-- /.description-block -->
                  </div>
                  <!-- /.col -->
                </div>
                <!-- /.row -->
              </div>
            </div>
            <!-- /.widget-user -->




      <div class="card">
              <div class="card-header p-2">
                <ul class="nav nav-pills">
                  <li class="nav-item"><a class="nav-link" href="#activity" data-toggle="tab">Activity</a></li>
                   <li class="nav-item"><a class="nav-link active show" href="#settings" data-toggle="tab">Settings</a></li>
                </ul>
              </div><!-- /.card-header -->
              <div class="card-body">
                <div class="tab-content">
                  <div class="tab-pane" id="activity">
                    <!-- Post -->
                   <h2>Activities here</h2>
                   </div>
                  <!-- /.tab-pane -->


                  <div class="tab-pane active show" id="settings">
                    <form class="form-horizontal">
                      <div class="form-group">
                        <label for="inputName" class="col-sm-2 control-label">Name</label>

                        <div class="col-sm-10">
                          <input type="email" v-model="form.name" class="form-control" id="inputName" placeholder="Name"
                          :class="{ 'is-invalid': form.errors.has('name') }">
                        <has-error :form="form" field="name"></has-error>
                        </div>
                      </div>
                      <div class="form-group">
                        <label for="inputEmail" class="col-sm-2 control-label">Email</label>

                        <div class="col-sm-10">
                          <input type="email" v-model="form.email" class="form-control" name="email" id="inputEmail" placeholder="Email"
                          :class="{ 'is-invalid': form.errors.has('email') }">
                           <has-error :form="form" field="email"></has-error>
                        </div>
                      </div>

                      <div class="form-group">
                        <label for="inputPassword" class="col-sm-6 control-label">Password (Leave empty if not changing)</label>

                        <div class="col-sm-10">
                        <input type="password" v-model="form.password" class="form-control" id="password" placeholder="Password"
                        :class="{ 'is-invalid': form.errors.has('password') }">
                        <has-error :form="form" field="password"></has-error>
                        </div>
                      </div>

                      <div class="form-group">
                        <label for="inputBio" class="col-sm-2 control-label">Biograpgy</label>

                        <div class="col-sm-10">
                          <textarea class="form-control" v-model="form.bio" id="inputBio" name="bio" placeholder="Biography"
                          :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                          <has-error :form="form" field="bio"></has-error>
                        </div>
                      </div>

                      <div class="form-group">
                        <label for="inputName2" class="col-sm-2 control-label">Profile Photo</label>

                        <div class="col-sm-10">
                          <input type="file" @change="updatePhoto" class="form-control" name="photo" id="inputFile" placeholder="Picture upload">
                        </div>
                      </div>




                      <div class="form-group">
                        <div class="col-sm-offset-2 col-sm-10">
                          <button type="submit" @click.prevent="updateInfo" class="btn btn-primary">Submit</button>
                        </div>
                      </div>
                    </form>
                  </div>
                  <!-- /.tab-pane -->
                </div>
                <!-- /.tab-content -->
              </div><!-- /.card-body -->
            </div>






            </div>
        </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                 form: new Form({
                    id: '',
                    name: '',
                    email: '',
                    password: '',
                    type: '',
                    bio: '',
                    photo: ''
                    })
            }
        },
        mounted() {
            console.log('Component mounted.')
        },
        methods: {
            getProfilePhoto(){
               let photo = (this.form.photo.length > 200) ? this.form.photo : "img/profile/"+ this.form.photo ;
                return photo;
            },
            updateInfo() {
                this.$Progress.start();
                if(this.form.password == ''){
                    this.form.password = undefined;
                }
                this.form.put('api/profile')
                .then(()=>{
                    Fire.$emit('EventAction');
                     // Display SweetAlert message dialog
                    toast.fire({
                    type: 'success',
                    title: 'Profile successfully updated'
                    })
                    this.$Progress.finish();
                })
                .catch(()=>{
                    this.$Progress.fail();
                });

            },
            updatePhoto (e){
                let file = e.target.files[0];
                let reader = new FileReader();
                let limit = 1024 * 1024 * 2;

                if(file['size'] > limit){
                    Swal.fire({
                        type: 'error',
                        title: 'Oops...',
                        text: 'You are uploading a large file',

                    })

                    return false;

                }
                reader.onloadend = (file) => {
                    this.form.photo = reader.result;
                }
                reader.readAsDataURL(file);
            }
        },
        created() {
            axios.get('api/profile')
            .then(({ data }) => (this.form.fill(data)));
        }
    }
</script>

<style scoped>
.widget-user-header{
    background-position: center center;
    background-size: cover;
    height: 200px;
}
</style>
