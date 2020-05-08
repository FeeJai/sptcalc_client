<template>
  <div class="container">
    <div class="row">
      <div class="col-sm-12">
        <h1>Substantial Presence Test - Calculator and Planner</h1>

        <h4 class="mb-2 mt-4">Current status</h4>
        <p v-for="year in futureYears" :key="year">
          {{year}}: {{compliantNonResident(year) ? 'Non Resident' : 'Tax Resident'}}
          <span
            v-if="compliantNonResident(year)"
          >for up to {{compliantDays(year)}} additional days</span>
          ({{daysIn(year)}} days present)
        </p>

        <h4 class="mb-2 mt-4">Past years</h4>
        <b-row class="mb-3">
          <b-col
            v-for="n in 3"
            :key="n"
          >{{currentYear - n}}: {{compliantNonResident(currentYear - n) ? 'Non Resident' : 'Tax Resident'}} ({{daysIn(currentYear - n)}} days)</b-col>
        </b-row>

        <h4 class="mb-2 mt-4">Planned stays</h4>
        <b-form-row>
          <b-col>
            <label for="range-1">Planned days in {{currentYear}}:</label>
          </b-col>
          <b-col>
            <b-form-input
              id="range-1"
              v-model="additionalDays.currentYear"
              type="range"
              min="0"
              :max="(365 - daysIn(currentYear, true))"
            ></b-form-input>
          </b-col>
          <b-col>
            <div>{{ additionalDays.currentYear }}/{{365 - daysIn(currentYear, true)}}</div>
          </b-col>
        </b-form-row>
        <b-form-row>
          <b-col>
            <label for="range-2">Planned days in {{currentYear + 1}}:</label>
          </b-col>
          <b-col>
            <b-form-input
              id="range-2"
              v-model="additionalDays.nextYear"
              type="range"
              min="0"
              :max="(365 - daysIn(currentYear + 1, true))"
            ></b-form-input>
          </b-col>
          <b-col>
            <div>{{ additionalDays.nextYear }}/{{365 - daysIn(currentYear + 1, true)}}</div>
          </b-col>
        </b-form-row>


        <h4 class="mb-2 mt-4">Travel history</h4>

        <b-row class="m-3">
          <button type="button" class="btn btn-success btn-sm" v-b-modal.trip-modal>Add Trip</button>
        </b-row>

        <table class="table table-hover">
          <thead>
            <tr>
              <th scope="col">Entry</th>
              <th scope="col">Exit</th>
              <th scope="col">Exempt?</th>
              <th></th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(trip, index) in trips" :key="index">
              <td>{{ trip.entry }}</td>
              <td>{{ trip.exit }}</td>
              <td>
                <span v-if="trip.exempt">Yes</span>
                <span v-else>No</span>
              </td>
              <td>
                <div class="btn-group" role="group">
                  <button
                    type="button"
                    class="btn btn-warning btn-sm"
                    v-b-modal.trip-update-modal
                    @click="editTrip(trip, index)"
                  >Update</button>
                  <button
                    type="button"
                    class="btn btn-danger btn-sm"
                    @click="onDeleteTrip(index)"
                  >Delete</button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <b-modal ref="addTripModal" id="trip-modal" entry="Add a new trip" hide-footer>
      <b-form @submit="onSubmit" @reset="onReset" class="w-100">
        <b-form-group id="form-entry-group" label="Entry:" label-for="form-entry-input">
          <b-form-datepicker
            id="form-entry-input"
            type="text"
            v-model="addTripForm.entry"
            required
            placeholder="Enter entry date"
            menu-class="w-100"
            calendar-width="100%"
          ></b-form-datepicker>
        </b-form-group>
        <b-form-group id="form-exit-group" label="Exit:" label-for="form-exit-input">
          <b-form-datepicker
            id="form-exit-input"
            type="text"
            v-model="addTripForm.exit"
            required
            placeholder="Enter exit date"
            menu-class="w-100"
            calendar-width="100%"
          ></b-form-datepicker>
        </b-form-group>
        <b-form-group id="form-exempt-group">
          <b-form-checkbox v-model="addTripForm.exempt" value="true">Exempt?</b-form-checkbox>
        </b-form-group>
        <b-button-group>
          <b-button type="submit" variant="primary">Add</b-button>
          <b-button type="reset" variant="danger">Cancel</b-button>
        </b-button-group>
      </b-form>
    </b-modal>

    <b-modal ref="editTripModal" id="trip-update-modal" entry="Update" hide-footer>
      <b-form @submit="onSubmitUpdate" @reset="onResetUpdate" class="w-100">
        <b-form-group id="form-entry-edit-group" label="Entry:" label-for="form-entry-edit-input">
          <b-form-datepicker
            id="form-entry-edit-input"
            type="text"
            v-model="editForm.entry"
            required
            placeholder="Enter entry date"
            menu-class="w-100"
            calendar-width="100%"
          ></b-form-datepicker>
        </b-form-group>
        <b-form-group id="form-exit-edit-group" label="Exit:" label-for="form-exit-edit-input">
          <b-form-datepicker
            id="form-exit-edit-input"
            type="text"
            v-model="editForm.exit"
            required
            placeholder="Enter exit date"
            menu-class="w-100"
            calendar-width="100%"
          ></b-form-datepicker>
        </b-form-group>
        <b-form-group id="form-exempt-edit-group">
          <b-form-checkbox v-model="editForm.exempt" value="true">Exempt?</b-form-checkbox>
        </b-form-group>
        <b-button-group>
          <b-button type="submit" variant="primary">Update</b-button>
          <b-button type="reset" variant="danger">Cancel</b-button>
        </b-button-group>
      </b-form>
    </b-modal>
  </div>
</template>

<script>
//import axios from "axios";
import moment from "moment";

export default {
  data() {
    return {
      currentYear: parseInt(moment().format("YYYY")),
      futureYears: [
        parseInt(moment().format("YYYY")) + 2,
        parseInt(moment().format("YYYY")) + 1,
        parseInt(moment().format("YYYY"))
      ],
      additionalDays: {
        currentYear: 0,
        nextYear: 0
      },
      trips: [
        {
          entry: "2020-01-01",
          exit: "2020-02-01",
          exempt: false
        },
        {
          entry: "2020-02-01",
          exit: "2020-02-01",
          exempt: true
        },
        {
          entry: "2019-01-01",
          exit: "2019-09-01",
          exempt: false
        }
      ],
      addTripForm: {
        entry: "",
        exit: "",
        exempt: []
      },
      message: "",
      showMessage: false,
      editForm: {
        id: "",
        entry: "",
        exit: "",
        exempt: []
      }
    };
  },
  components: {},
  computed: {},
  methods: {
    compliantNonResident: function(year) {
      /*       let currentYear = this.daysIn(year);
      if (
        currentYear < 31 ||
        currentYear + this.daysIn(year - 1) / 3 + this.daysIn(year - 2) / 6 <
          183
      )
        //there is no documentation on rounding in the official rules, therefore using floats
        return true;
      return false; */
      return this.compliantDays(year) >= 0;
    },
    compliantDays: function(year) {
      let currentYear = this.daysIn(year);
      let remaining = Math.max(
        31 - currentYear,
        183 -
          (currentYear + this.daysIn(year - 1) / 3 + this.daysIn(year - 2) / 6)
      );
      return Math.floor(remaining) - 1;
    },
    daysIn: function(year, excludePlanned = false) {
      let days = 0;
      let startOfYear = moment(year, "YYYY").startOf("year");
      let endOfYear = moment(year, "YYYY").endOf("year");
      if (year == this.currentYear && !excludePlanned)
        days += parseInt(this.additionalDays.currentYear);
      if (year == this.currentYear + 1 && !excludePlanned)
        days += parseInt(this.additionalDays.nextYear);
      for (let trip of this.trips) {
        if (trip.exempt) continue;
        let entryDate = moment(trip.entry);
        let exitDate = moment(trip.exit);
        if (entryDate <= endOfYear && exitDate >= startOfYear) {
          let begin = moment.max(entryDate, startOfYear);
          let end = moment.min(exitDate, endOfYear).add(1, "days"); //the day of entry and exit count, i.e. leaving on the same day is 1, not 0 days
          days += end.diff(begin, "days");
        }
      }
      return days;
    },
    initForm() {
      this.addTripForm.entry = "";
      this.addTripForm.exit = "";
      this.addTripForm.exempt = false;

      this.editForm.id = "";
      this.editForm.entry = "";
      this.editForm.exit = "";
      this.editForm.exempt = false;
    },
    onSubmit(evt) {
      evt.preventDefault();
      this.$refs.addTripModal.hide();
      let exempt = false;
      if (this.addTripForm.exempt[0]) exempt = true;
      const payload = {
        entry: this.addTripForm.entry,
        exit: this.addTripForm.exit,
        exempt
      };
      this.trips.push(payload);
      this.initForm();
    },
    onReset(evt) {
      evt.preventDefault();
      this.$refs.addTripModal.hide();
      this.initForm();
    },
    editTrip(trip, index) {
      this.editForm.id = index;
      this.editForm.entry = trip.entry;
      this.editForm.exit = trip.exit;
      this.editForm.exempt = trip.exempt;
    },
    onSubmitUpdate(evt) {
      evt.preventDefault();
      this.$refs.editTripModal.hide();
      console.log(this.editForm);
      let exempt = false;
      if (this.editForm.exempt) exempt = true;
      const payload = {
        entry: this.editForm.entry,
        exit: this.editForm.exit,
        exempt
      };
      this.trips[this.editForm.id] = payload;
      this.$forceUpdate();
    },
    onResetUpdate(evt) {
      evt.preventDefault();
      this.$refs.editTripModal.hide();
      this.initForm();
    },
    onDeleteTrip(trip) {
      this.trips.splice(trip, 1);
    }
  }
};
</script>