<template>
  <div class="container">
    <h5 class="fw-bold">Today Activity</h5>
    <div class="row">
      <div class="card bg-light mb-1">
        <div class="card-body">
          <div class="row">
            <div class="col-12 col-md-6 col-lg-6 my-1">
              <form class="row g-3" @submit.prevent="add_data">
                <input type="hidden" v-model="form.id">
                <input type="hidden" v-model="form.completed">
                <div class="col col-md-8 col-lg-8 d-flex align-items-center">
                  <label class="visually-hidden" for="input_activity">activity</label>
                  <input type="text" class="form-control form-control-sm" id="input_activity" placeholder="activity name" v-model="form.activity">
                </div>
                <div class="col col-md-2 col-lg-2 d-flex align-items-center">
                  <div class="form-check">
                    <input type="checkbox" class="form-check-input me-1" id="check_priority" v-model="form.priority">
                    <label class="form-check-label" for="check_priority">Priority</label>
                  </div>
                </div>
                <div class="col col-md-2 col-lg-2 d-flex align-items-center">
                  <button type="submit" class="btn btn-sm btn-secondary" v-show="!is_update_data"><i class="bi bi-plus-lg"></i></button>
                  <button type="button" class="btn btn-sm btn-secondary" v-show="is_update_data" @click="update_data(form)"><i class="bi bi-pencil-square"></i></button>
                  <button type="button" class="btn btn-sm btn-light ms-1" v-show="is_update_data" @click="load_data"><i class="bi bi-x-lg"></i></button>
                </div>
              </form>
            </div>
            <div class="col-12 col-md-6 col-lg-6 my-1">
              <div class="row">
                <div class="col d-flex align-items-center">
                  <span class="me-1">Search:</span><input type="text" class="form-control form-control-sm" id="input_search" placeholder="activity name" v-model="data_search" @keyup="on_search_data">
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <br>

    <h5>To Do <span class="badge bg-info">{{ to_do_list.length }} activity</span></h5>
    <div class="row" v-for="to_do in to_do_list" :key="to_do.id">
      <div class="card mb-1">
        <div class="card-body">
          <div class="row">
            <div class="col-6 d-flex align-items-center">
              <span><input type="checkbox" class="form-check-input me-1" @change="update_data_complete(to_do)"> {{ to_do.activity }}</span>
            </div>
            <div class="col-3 d-flex align-items-center">
              <span class="badge rounded-pill text-bg-danger w-auto p-2" v-if="to_do.priority === true"><i class="bi bi-bookmark-fill"></i> Priority</span>
              <span class="badge rounded-pill text-bg-light text-muted w-auto p-2" v-if="to_do.priority === false"><i class="bi bi-bookmark"></i> Priority</span>
            </div>
            <div class="col-3 d-flex align-items-center">
              <button type="button" class="btn btn-sm btn-secondary w-auto" @click="edit_data(to_do)"><i class="bi bi-pencil-square"></i></button>
              <button type="button" class="btn btn-sm btn-secondary w-auto ms-1" @click="delete_data(to_do)"><i class="bi bi-trash3"></i></button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <br>

    <h5>Complete <span class="badge bg-success">{{ to_do_list_complete.length }} activity</span></h5>
    <div class="row" v-for="to_do in to_do_list_complete" :key="to_do.id">
      <div class="card mb-1">
        <div class="card-body">
          <div class="row">
            <div class="col d-flex align-items-center text-success">
              <span><i class="bi bi-check-lg"></i> {{ to_do.activity }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import axios from 'axios'

  export default {
    name: 'TemplateHome',
    components: {},
    data() {
      return {
        form: {
          id: '', 
          activity: '',
          priority: '',
          completed: ''
        },
        to_do_list: [],
        to_do_list_complete: [],
        is_update_data: false
      };
    },
    mounted() {
      this.load_data();
    },
    methods: {
      // load to do list data
      load_data(avtivity_name) {
        this.form.id = ''; 
        this.form.activity = ''; 
        this.form.priority = '';
        this.form.completed = false;
        this.to_do_list = [];
        this.to_do_list_complete = [];
        this.is_update_data = false; 

        // http request
        var url = '';
        if (avtivity_name != undefined) {
          url = 'http://localhost:3000/to_do?activity_like=' + avtivity_name;
        } else {
          url = 'http://localhost:3000/to_do';
        }
        axios.get(url).then((res) => {
          res.data.forEach(data => {
            if (data.completed == true) {
              this.to_do_list_complete.push(data);
            } else {
              this.to_do_list.push(data);
            }
          });
        })
        .catch((err) => {
          console.log(err);
        });
      },

      // search to do list data by activity name
      on_search_data(e) {
        var avtivity_name = e.target.value;
        this.load_data(avtivity_name);
      },

      // add new to do list data
      add_data() {
        // validation
        if (this.form.activity == '') {
          this.show_alert_danger('Validation', 'Activity name cant be null.');
          return false;
        }
        if (this.form.priority == '') {
          this.form.priority = false;
        }

        // http request
        axios.post('http://localhost:3000/to_do/', this.form).then(() => {
          this.load_data();
          this.show_alert_success('Added', 'Your new activity has been added.');
        });
      },

      // init form edit with selected to do list data
      edit_data(to_do) {
        this.form.id = to_do.id; 
        this.form.activity = to_do.activity; 
        this.form.priority = to_do.priority;
        this.form.completed = to_do.completed;
        this.is_update_data = true; 
      },

      // update existing & selected to do list data
      update_data(form) {
        // validation
        if (this.form.activity == '') {
          this.show_alert_danger('Validation', 'Activity name cant be null.');
          return false;
        }

        var data_update = {
          activity: this.form.activity,
          priority: this.form.priority,
          completed: this.form.completed
        }
        // http request
        axios.put('http://localhost:3000/to_do/' + form.id, data_update).then(() => {
          this.load_data();
          this.show_alert_success('Updated', 'Your activity has been updated.');
        });
      },

      // update activity complete existing & selected to do list data
      update_data_complete(to_do) {
        var data_update = {
          activity: to_do.activity,
          priority: to_do.priority,
          completed: !to_do.completed
        }
        // http request
        axios.put('http://localhost:3000/to_do/' + to_do.id, data_update).then(() => {
          this.load_data();
          this.show_alert_success('Completed', 'Your activity has been completed.');
        });
      },

      // delete selected to do list data
      delete_data(to_do) {
        // delete activity confirmation
        this.$swal({
          text: 'Are you sure want to delete this activity?',
          type: 'warning',
          showCancelButton: true,
          confirmButtonColor: '#dc3545',
          confirmButtonText: 'Yes'
        }).then((result) => {
          if (result.value) {
            // http request
            axios.delete('http://localhost:3000/to_do/' + to_do.id).then(() => {
              this.load_data();
              this.$swal(
                'Deleted!',
                'Your activity has been deleted.',
                'success'
              );
            });
          }
        });
      },

      // sweetalert2 success show
      show_alert_success(message_title, message_body) {
        this.$swal(
          message_title,
          message_body,
          'success'
        );
      },

      // sweetalert2 error show
      show_alert_danger(message_title, message_body) {
        this.$swal(
          message_title,
          message_body,
          'error'
        );
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
