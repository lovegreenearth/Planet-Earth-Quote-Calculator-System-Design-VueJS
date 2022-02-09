<template>
<div class="container">
  <div class="title">
    <h2>Settings</h2>
  </div>
  <div class="content">
    <div id="settings-rooms">
      <form id="rooms-page">
        
          <div class="row text-l">
            <div class="col-3 text-l">
              <label class="text-l">Rooms:</label>
            </div>
            <div class="col-9">
              <custom-dropdown
                :options="rooms"
                :default="'Edit rooms'"
                class="select"
                v-model="selected"
                @onAdd="addRoom($event)"
                @onDelete="confirmDeleteRoom($event)"
                @onEdit="editRoom($event)"
              />
            </div>
          </div>
        
          <div class="row h-space">
            <div class="col-3 h-space">
              <label>Activities</label>
            </div>
            <div class="col-9 h-space">
              <custom-dropdown
                :options="activitiesDropdown"
                :default="'Edit an activity'"
                class="select"
                v-model="selectedActivity"
                keyName="name"
                keyValue="id"
                @onDelete="confirmDeleteActivity($event)" 
                @onAdd="addActivity($event)"
                @onSelect="clearFields()"
              />
                <!-- @onDelete="confirmDeleteRoom($event)"
                @onEdit="editRoom($event)" -->
            </div>
          </div>
      </form>
    </div>
  </div>
</div>
  
</template>

<script>
import request from '@/helpers/fetchWrapper'
import { GET_TAXONOMY, UPDATE_TAXONOMY } from '@/utils/const'
import CustomDropdown from '../../components/common/CustomDropdown.vue';

export default {
  name: 'SettingsRooms',
  components: {
    CustomDropdown
  },
  props: {
    msg: String,
    activitiesDropdown: [],
  },
  mounted() {
    this.fetchRooms();
    this.fetchActivities();
  },
  // computed: {
  //   currentProposal () {
  //     return this.$store.state.currentProposal;
  //   },
  //   propoosal () {
  //     return this.$store.state.currentProposal;
  //   },
  //   globalLevels () {
  //     return this.$store.state.levels;
  //   },
  //   currentProposalLevel () {
  //     return this.$store.state.currentProposalLevel;
  //   },
  //   selectedLevel() {
  //      return this.currentProposalLevel?.level?.name || "Select level here"
  //   },
  //   ...mapGetters([
  //     'currentProposalLevelIndex',
  //     'currentProposalRoomIndex',

  //   ]),
  // },
  data() {
    return {
      showAddRoom: false,
      weeks: [
        "Week 1",
        "Week 2",
        "Week 3",
        "Week 4"
      ],
      check_weeks: [],
      action: 'add',
      rooms:[],
      selected: null,
      selectedActivity: "",
      roomData: {
        name:'',
        description:''
      },

    };
  },
  methods: {
    fetchRooms: function(){
        request(`${GET_TAXONOMY}rooms?per_page=99`, {
          method: 'GET'
        }).then((res) => {              
            this.rooms = res
        }).catch((err) => {
          console.log(err)
        })
    },
    fetchActivities: function () {
      request(`${GET_TAXONOMY}activities?per_page=99`, {
        method: "GET",
      })
        .then((res) => {
          console.log(res);
          this.activitiesDropdown = res;
        })
        .catch((err) => {
          console.log(err);
        });
    },
    confirmDeleteActivity: function(e) {

      const currentActivityId = e;
      // const findActivity = this.activity.find(a=> a.activity.term_id === this.selectedActivity.id ||  a.activity.id === this.selectedActivity.id);
      // const currActivityName = findActivity.name;

       this.$confirm(
        {
          message: `Are you sure you want to delete the activity?`, //${currActivityName}
          button: {
            no: 'No',
            yes: 'Yes'
          },
          /**
          * Callback Function
          * @param {Boolean} confirm 
          */
          callback: confirm => {
            if (confirm) {
              this.deleteActivity(currentActivityId)
            }
          }
        }
      )
    },
    addActivity: function (e) {
      this.action = "add";
      const activityData = {
        name: e,
        description: "",
      };

      if (activityData.name !== "") this.saveActivityData(activityData);
    },
    clearFields: function () {
    
    },
    saveActivityData: function (activityData) {
      const url =
        this.action === "add"
          ? `${UPDATE_TAXONOMY}activities`
          : `${UPDATE_TAXONOMY}activities/${this.settings.ID}`; 
          // find id first

      let formData = activityData;

      request(`${url}`, {
        method: "POST",
        body: JSON.stringify(formData),
        headers: {
          "Content-Type": "application/json",
        },
      })
        .then((res) => {
          console.log(res);
          this.fetchActivities();
        })
        .catch((err) => {
          console.log(err);
        });
    },
    saveData: function() {
      const url = this.action==='add' ?  `${UPDATE_TAXONOMY}rooms`  : `${UPDATE_TAXONOMY}rooms/${this.id}`
      
      let formData = {...this.roomData};

      request(`${url}`, {
        method: 'POST',
        body: JSON.stringify(formData),
        headers: {
          'Content-Type': 'application/json'
        },
      }).then((res) => {
        console.log(res);
        this.roomData.name = ''
        this.showAddRoom = false
        this.fetchRooms();
      }).catch((err) => {
        console.log(err)
      })
    },
    confirmDeleteRoom: function(e) {

      const currentRoomId = e;
      const findRoom = this.rooms.find(a => a.id === e);
      const currRoomName = findRoom.name;

      // if (this.selected === null) return

       this.$confirm(
        {
          message: `Are you sure you want to delete ${currRoomName}?`,
          button: {
            no: 'No',
            yes: 'Yes'
          },
          /**
          * Callback Function
          * @param {Boolean} confirm 
          */
          callback: confirm => {
            if (confirm) {
              this.deleteRoom(currentRoomId);
            }
          }
        }
      )
    },

    toggleForm: function(action){
      this.action = action;
      this.showAddRoom = !this.showAddRoom
    },
    addRoom: function(e){
      this.action = 'add';

      // this.roomData.name = this.$refs.room_select.$data.searchText;
      this.roomData.name = e;

      const findRoom = this.rooms.find(a => a.name === this.roomData.name)
      if (this.roomData.name !== '' && findRoom === undefined){
        this.saveData()
      }
      
    },
    editRoom: function(e = null){
      const name = e.value || this.selected.name;

      this.action = 'edit';
      this.roomData.name = name;
      this.id = e.id;

      this.saveData();
    },
    deleteRoom: function(id){
      // if (this.selected === null) return
      request( `${UPDATE_TAXONOMY}rooms/${id}`, {
          method: 'DELETE'
        }).then((res) => {              
            console.log(`${UPDATE_TAXONOMY}rooms/${id}`)
            this.fetchRooms()
        }).catch((err) => {
          console.log(err)
        })
    }

  }

};
</script>

<style lang="scss" >
  // settings-rooms
  
  
</style>