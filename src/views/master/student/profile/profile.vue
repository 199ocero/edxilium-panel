<template>
  <div class="layout-px-spacing">
    <portal to="breadcrumb">
      <ul class="navbar-nav flex-row">
        <li>
          <div class="page-header">
            <nav class="breadcrumb-one" aria-label="breadcrumb">
              <ol class="breadcrumb">
                <li class="breadcrumb-item active" aria-current="page">
                  <span>Student - Profile</span>
                </li>
              </ol>
            </nav>
          </div>
        </li>
      </ul>
    </portal>

    <div class="row layout-spacing">
      <!-- Content -->
      <div class="col-xl-12 col-lg-12 col-md-12 col-sm-12 layout-top-spacing">
        <div class="seperator-header">
          <h4>Student Profile</h4>
        </div>
        <div class="panel">
          <div class="panel-body">
            <b-form-row class="mb-4">
              <b-form-group label="First Name" class="col-md-4">
                <b-input v-model="form.first_name1" name="first_name" type="text" placeholder="First Name" readonly></b-input>
              </b-form-group>
              <b-form-group label="Middle Name" class="col-md-4">
                <b-input v-model="form.middle_name1" name="middle_name" type="text" placeholder="Middle Name" readonly></b-input>
              </b-form-group>
              <b-form-group label="Last Name" class="col-md-4">
                <b-input v-model="form.last_name1" name="last_name" type="text" placeholder="Last Name" readonly></b-input>
              </b-form-group>
            </b-form-row>
            <b-form-row class="mb-4">
              <b-form-group label="Age" class="col-md-4" readonly>
                <b-input v-model="form.age1" name="age" v-mask="'##'" placeholder="__" readonly></b-input>
              </b-form-group>
              <b-form-group label="Gender" class="col-md-4">
                <b-input v-model="form.gender1" name="gender" type="text" placeholder="Gender" readonly></b-input>
              </b-form-group>
              <b-form-group label="Contact Number" class="col-md-4">
                <b-input v-model="form.contact_number1" v-mask="'(###) ###-####'" placeholder="(___) ___-____" readonly></b-input>
                <small class="form-text text-muted"><span style="color: #009688">Note:</span> Contact number should start with number 9.</small>
              </b-form-group>
            </b-form-row>
            <b-form-row class="mb-4">
              <b-form-group label="Facebook ID" class="col-md-12">
                <b-input v-model="form.facebook_id1" v-mask="'################'" placeholder="________________" readonly></b-input>
                <small class="form-text text-muted"><span style="color: #009688">Note:</span> Facebook ID is a 16 digit numbers.</small>
              </b-form-group>
            </b-form-row>
            <hr />
            <div class="d-flex flex-wrap justify-content-center justify-content-sm-end">
              <b-button type="submit" variant="primary" class="mt-3 m-1" v-b-modal.studentEditModal @click="editStudent">Edit Profile</b-button>
            </div>
          </div>
        </div>
      </div>
    </div>
    <vue-progress-bar></vue-progress-bar>
    <!--Edit Modal for Student -->
    <b-modal id="studentEditModal" title="Edit Profile" centered size="lg" hide-footer @hidden="editResetModal">
      <b-form action="#" @submit.prevent="updateStudent" @keydown="errors.clear($event.target.name)">
        <b-form-row class="mb-4">
          <b-form-group label="First Name" class="col-md-4">
            <b-input v-model="form.first_name" name="first_name" type="text" placeholder="First Name"></b-input>
            <span class="text-danger" v-text="errors.get('first_name')"></span>
          </b-form-group>
          <b-form-group label="Middle Name" class="col-md-4">
            <b-input v-model="form.middle_name" name="middle_name" type="text" placeholder="Middle Name"></b-input>
            <span class="text-danger" v-text="errors.get('middle_name')"></span>
          </b-form-group>
          <b-form-group label="Last Name" class="col-md-4">
            <b-input v-model="form.last_name" name="last_name" type="text" placeholder="Last Name"></b-input>
            <span class="text-danger" v-text="errors.get('last_name')"></span>
          </b-form-group>
        </b-form-row>
        <b-form-row class="mb-4">
          <b-form-group label="Age" class="col-md-4">
            <b-input v-model="form.age" name="age" v-mask="'##'" placeholder="__"></b-input>
            <span class="text-danger" v-text="errors.get('age')"></span>
          </b-form-group>
          <b-form-group label="Gender" class="col-md-4">
            <!-- Default Select -->
            <b-select v-model="form.gender">
              <b-select-option :value="null" disabled>Select Gender</b-select-option>
              <b-select-option value="Male">Male</b-select-option>
              <b-select-option value="Female">Female</b-select-option>
            </b-select>
            <span class="text-danger" v-text="errors.get('gender')"></span>
          </b-form-group>
          <b-form-group label="Contact Number" class="col-md-4">
            <b-input name="contact_number" v-model="form.contact_number" v-mask="'(###) ###-####'" placeholder="(___) ___-____"></b-input>
            <small class="form-text text-muted"><span style="color: #009688">Note:</span> Contact number should start with number 9.</small>
            <span class="text-danger" v-text="errors.get('contact_number')"></span>
          </b-form-group>
        </b-form-row>
        <b-form-row class="mb-4">
          <b-form-group label="Facebook ID" class="col-md-12">
            <b-input v-model="form.facebook_id" v-mask="'################'" placeholder="________________"></b-input>
            <small class="form-text text-muted"><span style="color: #009688">Note:</span> Facebook ID is a 16 digit numbers.</small>
          </b-form-group>
        </b-form-row>

        <hr />
        <div class="d-flex flex-wrap justify-content-center justify-content-sm-end">
          <b-button type="submit" variant="primary" class="mt-3 m-1">Update</b-button>
          <b-button variant="danger" class="mt-3 m-1" @click="$bvModal.hide('studentEditModal')">Cancel</b-button>
        </div>
      </b-form>
    </b-modal>
  </div>
</template>

<script>
import Vue from 'vue';
import VueMask from 'v-mask';
Vue.use(VueMask);
import Errors from '@/main.js';
import '@/assets/sass/scrollspyNav.scss';
import '@/assets/sass/users/user-profile.scss';
export default {
  metaInfo: { title: 'Student Profile' },
  data() {
    return {
      form: {
        first_name: '',
        middle_name: '',
        last_name: '',
        age: '',
        gender: '',
        contact_number: '',
        email: '',
        facebook_id: '',
        first_name1: '',
        middle_name1: '',
        last_name1: '',
        age1: '',
        gender1: '',
        contact_number1: '',
        email1: '',
        facebook_id1: '',
      },
      errors: new Errors(),
    };
  },
  mounted() {
    this.bind_data();
  },
  methods: {
    bind_data() {
      this.$http
        .get('/api/student/info/profile', {
          headers: {
            Authorization: `Bearer ${localStorage.getItem('token')}`,
          },
        })
        .then((response) => {
          this.form.first_name1 = response.data.data.first_name;
          this.form.middle_name1 = response.data.data.middle_name;
          this.form.last_name1 = response.data.data.last_name;
          this.form.age1 = response.data.data.age;
          this.form.gender1 = response.data.data.gender;
          this.form.contact_number1 = response.data.data.contact_number;
          this.form.facebook_id1 = response.data.data.facebook_id;
        })
        .catch((errors) => {
          this.errors.record(errors.response.data.errors);
        });
    },
    editStudent() {
      // console.log(id);
      this.$http
        .get('/api/student/info/profile', {
          headers: {
            Authorization: `Bearer ${localStorage.getItem('token')}`,
          },
        })
        .then((response) => {
          this.form.first_name = response.data.data.first_name;
          this.form.middle_name = response.data.data.middle_name;
          this.form.last_name = response.data.data.last_name;
          this.form.age = response.data.data.age;
          this.form.gender = response.data.data.gender;
          this.form.contact_number = response.data.data.contact_number;
          this.form.facebook_id = response.data.data.facebook_id;
        })
        .catch((errors) => {
          this.errors.record(errors.response.data.errors);
        });
    },
    updateStudent() {
      let self = this;
      var axios = require('axios');
      var data = this.form;
      var config = {
        method: 'put',
        url: '/api/student/info/profile',
        headers: {
          Authorization: 'Bearer ' + localStorage.getItem('token'),
        },
        data: data,
      };
      self.$Progress.start();
      axios(config)
        .then(function () {
          self.$toaster.success('Profile Update Successfuly!');
          self.$nextTick(() => {
            self.$bvModal.hide('studentEditModal');
          });
          self.bind_data();
          self.$Progress.finish();
        })
        .catch(function (errors) {
          self.errors.record(errors.response.data.errors);
          self.$Progress.fail();
        });
    },
    editResetModal() {
      this.form.first_name = 'first_name';
      this.form.middle_name = 'middle_name';
      this.form.last_name = 'last_name';
      this.form.age = 'age';
      this.form.gender = 'gender';
      this.form.contact_number = 'contact_number';
      this.form.facebook_id = 'facebook_id';

      this.form.first_name = '';
      this.form.middle_name = '';
      this.form.last_name = '';
      this.form.age = '';
      this.form.gender = '';
      this.form.contact_number = '';
      this.form.facebook_id = '';

      this.errors.clear('first_name');
      this.errors.clear('middle_name');
      this.errors.clear('last_name');
      this.errors.clear('age');
      this.errors.clear('gender');
      this.errors.clear('contact_number');
      this.errors.clear('facebook_id');
    },
  },
};
</script>
