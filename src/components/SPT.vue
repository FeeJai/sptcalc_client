<template>
  <div class="container">
    <div class="row">
      <div class="col-sm-12">
        <h1>Substantial Presence Test - Calculator and Planner</h1>

        <h4 class="mb-2 mt-4">Current status</h4>
        <p v-for="year in futureYears" :key="year">
          {{year}}: {{isCompliantNonResident(year) ? 'Non Resident' : 'Tax Resident'}}
          <span
            v-if="isCompliantNonResident(year)"
          >for up to {{compliantDays(year)}} additional days</span>
          ({{daysPresent(year)}} days present)
        </p>

        <h4 class="mb-2 mt-4">Past years</h4>
        <b-row class="mb-3">
          <b-col
            v-for="n in 3"
            :key="n"
          >{{currentYear - n}}: {{isCompliantNonResident(currentYear - n) ? 'Non Resident' : 'Tax Resident'}} ({{daysPresent(currentYear - n)}} days)</b-col>
        </b-row>

        <h4 class="mb-2 mt-4">Planned stays</h4>
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
              :max="(daysPerYear(currentYear + 1) - daysPresent(currentYear + 1, true))"
            ></b-form-input>
          </b-col>
          <b-col>
            <div>{{ additionalDays.nextYear }}/{{daysPerYear(currentYear + 1) - daysPresent(currentYear + 1, true)}}</div>
          </b-col>
        </b-form-row>

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
              :max="(daysPerYear(currentYear) - daysPresent(currentYear, true))"
            ></b-form-input>
          </b-col>
          <b-col>
            <div>{{ additionalDays.currentYear }}/{{daysPerYear(currentYear) - daysPresent(currentYear, true)}}</div>
          </b-col>
        </b-form-row>

        <h4 class="mb-2 mt-4">Travel history</h4>

        <b-row class="m-1">
          <button type="button" class="btn btn-success m-1" v-b-modal.trip-modal>Add Trip</button>
          <button
            type="button"
            class="btn btn-primary m-1"
            v-b-modal.download-modal
          >Download Travel History from CBP</button>
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
            <tr v-for="(trip, index) in trips" :key="index" :class="rowClass(index)">
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

    <b-modal id="download-modal" ref="downloadModal" title="Download Travel History" hide-footer>
      <p class="my-4">Download i94 Travel History from CBP:</p>
      <b-container fluid>
        <b-form @submit="onDownload" class="w-100">
          <b-form-row class="mb-2">
            <b-col sm="3">
              <label for="form-lastName-input" class="mr-2">Name:</label>
            </b-col>
            <b-col sm="4">
              <b-form-input
                id="form-lastName-input"
                type="text"
                v-model="downloadData.lastName"
                required
                placeholder="Last"
              ></b-form-input>
            </b-col>
            <b-col sm="4">
              <b-form-input
                id="form-firstName-input"
                type="text"
                v-model="downloadData.firstName"
                required
                placeholder="First"
              ></b-form-input>
            </b-col>
          </b-form-row>
          <b-form-row class="mb-2">
            <b-col sm="3">
              <label for="form-birthYear-input" class="mr-2">DOB:</label>
            </b-col>
            <b-col sm="3">
              <b-form-input
                id="form-birthYear-input"
                type="text"
                v-model="downloadData.birthYear"
                required
                placeholder="YYYY"
              ></b-form-input>
            </b-col>
            <b-col sm="2">
              <b-form-input
                id="form-birthMonth-input"
                type="text"
                v-model="downloadData.birthMonth"
                required
                placeholder="MM"
              ></b-form-input>
            </b-col>
            <b-col sm="2">
              <b-form-input
                id="form-birthDay-input"
                type="text"
                v-model="downloadData.birthDay"
                required
                placeholder="DD"
              ></b-form-input>
            </b-col>
          </b-form-row>
          <b-form-row class="mb-2">
            <b-col sm="3">
              <label for="form-lastName-input" class="mr-2">Country:</label>
            </b-col>
            <b-col sm="8">
              <b-form-select v-model="downloadData.passportCountry" :options="countries"></b-form-select>
            </b-col>
          </b-form-row>
          <b-form-row class="mb-2">
            <b-col sm="3">
              <label for="form-passportNumber-input" class="mr-2">Passport No:</label>
            </b-col>
            <b-col sm="8">
              <b-form-input
                id="form-passportNumber-input"
                type="text"
                v-model="downloadData.passportNumber"
                required
                placeholder="Passport Number"
              ></b-form-input>
            </b-col>
          </b-form-row>
          <b-button-group>
            <b-button type="submit" variant="primary">Download</b-button>
          </b-button-group>
        </b-form>
      </b-container>
    </b-modal>

    <b-modal ref="addTripModal" id="trip-modal" entry="Add a new trip" hide-footer>
      <b-form @submit="onAddTrip" @reset="onCancelAdding" class="w-100">
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
      <b-form @submit="onEdit" @reset="onCancelEdit" class="w-100">
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
import axios from "axios";
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
      trips: [],
      downloadData: {
        errorMsg: ""
      },
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
      },
      countries: [
        { value: "AFG", text: "Afghanistan" },
        { value: "ALB", text: "Albania" },
        { value: "DZA", text: "Algeria" },
        { value: "ASM", text: "American Samoa" },
        { value: "AND", text: "Andorra" },
        { value: "AGO", text: "Angola" },
        { value: "AIA", text: "Anguilla" },
        { value: "ATA", text: "Antarctica" },
        { value: "ATG", text: "Antigua and Barbuda" },
        { value: "ARG", text: "Argentina" },
        { value: "ARM", text: "Armenia" },
        { value: "ABW", text: "Aruba" },
        { value: "AUS", text: "Australia" },
        { value: "AUT", text: "Austria" },
        { value: "AZE", text: "Azerbaijan" },
        { value: "BHS", text: "Bahamas, The" },
        { value: "BHR", text: "Bahrain" },
        { value: "BGD", text: "Bangladesh" },
        { value: "BRB", text: "Barbados" },
        { value: "BLR", text: "Belarus" },
        { value: "BEL", text: "Belgium" },
        { value: "BLZ", text: "Belize" },
        { value: "BEN", text: "Benin" },
        { value: "BMU", text: "Bermuda" },
        { value: "BTN", text: "Bhutan" },
        { value: "BOL", text: "Bolivia" },
        { value: "BES", text: "Bonaire, Sint Eustatius, and Saba" },
        { value: "BIH", text: "Bosnia and Herzegovina" },
        { value: "BWA", text: "Botswana" },
        { value: "BVT", text: "Bouvet Island" },
        { value: "BRA", text: "Brazil" },
        { value: "OIT", text: "British Indian Ocean Territory" },
        { value: "VGB", text: "British Virgin Islands" },
        { value: "BRN", text: "Brunei" },
        { value: "BGR", text: "Bulgaria" },
        { value: "BFA", text: "Burkina Faso" },
        { value: "MMR", text: "Burma (Myanmar)" },
        { value: "BDI", text: "Burundi" },
        { value: "KHM", text: "Cambodia" },
        { value: "CMR", text: "Cameroon" },
        { value: "CAN", text: "Canada" },
        { value: "CTE", text: "Canton and Enderbury Islands" },
        { value: "CPV", text: "Cape Verde" },
        { value: "CYM", text: "Cayman Islands" },
        { value: "CAF", text: "Central African Republic" },
        { value: "TCD", text: "Chad" },
        { value: "CHL", text: "Chile" },
        { value: "CHN", text: "China" },
        { value: "CXR", text: "Christmas Island" },
        { value: "CCK", text: "Cocos (Keeling) Islands" },
        { value: "COL", text: "Colombia" },
        { value: "COM", text: "Comoros" },
        { value: "COD", text: "Congo, Democratic Republic of the" },
        { value: "COG", text: "Congo, Republic of the" },
        { value: "COK", text: "Cook Islands" },
        { value: "CRI", text: "Costa Rica" },
        { value: "CIV", text: "Cote d'Ivoire" },
        { value: "HRV", text: "Croatia" },
        { value: "CUB", text: "Cuba" },
        { value: "CUW", text: "Curacao" },
        { value: "CYP", text: "Cyprus" },
        { value: "CZE", text: "Czech Republic" },
        { value: "YMD", text: "Democratic Yemen" },
        { value: "DNK", text: "Denmark" },
        { value: "DJI", text: "Djibouti" },
        { value: "DMA", text: "Dominica" },
        { value: "DOM", text: "Dominican Republic" },
        { value: "DML", text: "Dronning Maud Land" },
        { value: "TMP", text: "East Timor" },
        { value: "ECU", text: "Ecuador" },
        { value: "EGY", text: "Egypt" },
        { value: "SLV", text: "El Salvador" },
        { value: "GNQ", text: "Equatorial Guinea" },
        { value: "ERI", text: "Eritrea" },
        { value: "EST", text: "Estonia" },
        { value: "ETH", text: "Ethiopia" },
        { value: "FLK", text: "Falkland Islands (Islas Malvinas)" },
        { value: "FRO", text: "Faroe Islands" },
        { value: "FJI", text: "Fiji" },
        { value: "FIN", text: "Finland" },
        { value: "FRA", text: "France" },
        { value: "FXX", text: "France, Metropolitan" },
        { value: "GUF", text: "French Guiana" },
        { value: "PYF", text: "French Polynesia" },
        { value: "GAB", text: "Gabon" },
        { value: "GMB", text: "Gambia, The" },
        { value: "GGI", text: "Georgia" },
        { value: "NTD", text: "Georgia Status Neutral Travel Document" },
        { value: "DEU", text: "Germany" },
        { value: "GHA", text: "Ghana" },
        { value: "GIB", text: "Gibraltar" },
        { value: "GRC", text: "Greece" },
        { value: "GRL", text: "Greenland" },
        { value: "GRD", text: "Grenada" },
        { value: "GLP", text: "Guadeloupe" },
        { value: "GUM", text: "Guam" },
        { value: "GTM", text: "Guatemala" },
        { value: "GIN", text: "Guinea" },
        { value: "GNB", text: "Guinea-Bissau" },
        { value: "GUY", text: "Guyana" },
        { value: "HTI", text: "Haiti" },
        { value: "HMD", text: "Heard Island and McDonald Islands" },
        { value: "VAT", text: "Holy See (Vatican City)" },
        { value: "HND", text: "Honduras" },
        { value: "HKG", text: "Hong Kong" },
        { value: "HUN", text: "Hungary" },
        { value: "ISL", text: "Iceland" },
        { value: "IND", text: "India" },
        { value: "IDN", text: "Indonesia" },
        { value: "IRN", text: "Iran" },
        { value: "IRQ", text: "Iraq" },
        { value: "IRL", text: "Ireland" },
        { value: "IMN", text: "Isle of Man" },
        { value: "ISR", text: "Israel" },
        { value: "ITA", text: "Italy" },
        { value: "JAM", text: "Jamaica" },
        { value: "JPN", text: "Japan" },
        { value: "JOR", text: "Jordan" },
        { value: "KAZ", text: "Kazakhstan" },
        { value: "KEN", text: "Kenya" },
        { value: "KIR", text: "Kiribati" },
        { value: "PRK", text: "Korea, North" },
        { value: "KOR", text: "Korea, South" },
        { value: "RKS", text: "Kosovo" },
        { value: "KWT", text: "Kuwait" },
        { value: "KGY", text: "Kyrgyzstan" },
        { value: "LAO", text: "Laos" },
        { value: "LVA", text: "Latvia" },
        { value: "LBN", text: "Lebanon" },
        { value: "LSO", text: "Lesotho" },
        { value: "LBR", text: "Liberia" },
        { value: "LBY", text: "Libya" },
        { value: "LIE", text: "Liechtenstein" },
        { value: "LTU", text: "Lithuania" },
        { value: "LUX", text: "Luxembourg" },
        { value: "MAC", text: "Macau" },
        { value: "MKD", text: "Macedonia" },
        { value: "MDG", text: "Madagascar" },
        { value: "MWI", text: "Malawi" },
        { value: "MYS", text: "Malaysia" },
        { value: "MDV", text: "Maldives" },
        { value: "MLI", text: "Mali" },
        { value: "MLT", text: "Malta" },
        { value: "MHL", text: "Marshall Islands" },
        { value: "MTQ", text: "Martinique" },
        { value: "MRT", text: "Mauritania" },
        { value: "MUS", text: "Mauritius" },
        { value: "MYT", text: "Mayotte" },
        { value: "MEX", text: "Mexico" },
        { value: "FSM", text: "Micronesia, Federated States of" },
        { value: "MOA", text: "Moldova" },
        { value: "MCO", text: "Monaco" },
        { value: "MNG", text: "Mongolia" },
        { value: "MNE", text: "Montenegro" },
        { value: "MSR", text: "Montserrat" },
        { value: "MAR", text: "Morocco" },
        { value: "MOZ", text: "Mozambique" },
        { value: "NAM", text: "Namibia" },
        { value: "NRU", text: "Nauru" },
        { value: "NPL", text: "Nepal" },
        { value: "NLD", text: "Netherlands" },
        { value: "ANT", text: "Netherlands Antilles" },
        { value: "NCL", text: "New Caledonia" },
        { value: "NZL", text: "New Zealand" },
        { value: "NIC", text: "Nicaragua" },
        { value: "NER", text: "Niger" },
        { value: "NGA", text: "Nigeria" },
        { value: "NIU", text: "Niue" },
        { value: "NFK", text: "Norfolk Island" },
        { value: "MNP", text: "Northern Mariana Islands" },
        { value: "NOR", text: "Norway" },
        { value: "OMN", text: "Oman" },
        { value: "PAK", text: "Pakistan" },
        { value: "PLW", text: "Palau" },
        { value: "PSE", text: "Palestine" },
        { value: "PAN", text: "Panama" },
        { value: "PNG", text: "Papua New Guinea" },
        { value: "PRY", text: "Paraguay" },
        { value: "PER", text: "Peru" },
        { value: "PHL", text: "Philippines" },
        { value: "PCN", text: "Pitcairn Islands" },
        { value: "POL", text: "Poland" },
        { value: "PRT", text: "Portugal" },
        { value: "PRI", text: "Puerto Rico" },
        { value: "QAT", text: "Qatar" },
        { value: "REU", text: "Reunion" },
        { value: "ROU", text: "Romania" },
        { value: "RUS", text: "Russia" },
        { value: "RWA", text: "Rwanda" },
        { value: "BLM", text: "Saint Barthelemy" },
        { value: "SHN", text: "Saint Helena" },
        { value: "KNA", text: "Saint Kitts and Nevis" },
        { value: "LCA", text: "Saint Lucia" },
        { value: "MAF", text: "Saint Martin" },
        { value: "SPM", text: "Saint Pierre and Miquelon" },
        { value: "VCT", text: "Saint Vincent and the Grenadines" },
        { value: "WSM", text: "Samoa" },
        { value: "SMR", text: "San Marino" },
        { value: "STP", text: "Sao Tome and Principe" },
        { value: "SAU", text: "Saudi Arabia" },
        { value: "SEN", text: "Senegal" },
        { value: "SBA", text: "Serbia" },
        { value: "SYC", text: "Seychelles" },
        { value: "SLE", text: "Sierra Leone" },
        { value: "SGP", text: "Singapore" },
        { value: "SXM", text: "Sint Maarten" },
        { value: "SVK", text: "Slovakia" },
        { value: "SVN", text: "Slovenia" },
        { value: "SLB", text: "Solomon Islands" },
        { value: "SOM", text: "Somalia" },
        { value: "ZAF", text: "South Africa" },
        { value: "SSD", text: "South Sudan" },
        { value: "ESP", text: "Spain" },
        { value: "LKA", text: "Sri Lanka" },
        { value: "SDN", text: "Sudan" },
        { value: "SUR", text: "Suriname" },
        { value: "SJM", text: "Svalbard" },
        { value: "SWR", text: "Swaziland" },
        { value: "SWE", text: "Sweden" },
        { value: "CHE", text: "Switzerland" },
        { value: "SYR", text: "Syria" },
        { value: "TWN", text: "Taiwan" },
        { value: "TJK", text: "Tajikistan" },
        { value: "TZA", text: "Tanzania" },
        { value: "THA", text: "Thailand" },
        { value: "TLS", text: "Timor-Leste" },
        { value: "TGO", text: "Togo" },
        { value: "TKL", text: "Tokelau" },
        { value: "TON", text: "Tonga" },
        { value: "TTO", text: "Trinidad and Tobago" },
        { value: "TUN", text: "Tunisia" },
        { value: "TUR", text: "Turkey" },
        { value: "TKM", text: "Turkmenistan" },
        { value: "TCA", text: "Turks and Caicos Islands" },
        { value: "TUV", text: "Tuvalu" },
        { value: "UGA", text: "Uganda" },
        { value: "UKR", text: "Ukraine" },
        { value: "ARE", text: "United Arab Emirates" },
        { value: "GBR", text: "United Kingdom" },
        { value: "USA", text: "United States" },
        { value: "UMI", text: "United States Minor Outlying Islands" },
        { value: "URY", text: "Uruguay" },
        { value: "UZB", text: "Uzbekistan" },
        { value: "VUT", text: "Vanuatu" },
        { value: "VEN", text: "Venezuela" },
        { value: "VNM", text: "Vietnam" },
        { value: "VIR", text: "Virgin Islands" },
        { value: "WAK", text: "Wake Island" },
        { value: "WLF", text: "Wallis and Futuna" },
        { value: "ESH", text: "Western Sahara" },
        { value: "YEM", text: "Yemen" },
        { value: "ZAR", text: "Zaire" },
        { value: "ZMB", text: "Zambia" },
        { value: "ZWE", text: "Zimbabwe" }
      ]
    };
  },
  components: {},
  computed: {},
  methods: {
    rowClass(index) {
      let entry = moment(this.trips[index].entry);
      let exit = moment(this.trips[index].exit);
      if (!entry.isValid() || !exit.isValid()) return "table-warning";
      if (entry.isAfter(exit)) return "table-danger";
      if (
        typeof this.trips[index - 1] !== "undefined" &&
        exit.isAfter(moment(this.trips[index - 1].entry))
      )
        return "table-info";
      if (
        typeof this.trips[index + 1] !== "undefined" &&
        entry.isBefore(moment(this.trips[index + 1].exit))
      )
        return "table-info";
    },
    onDownload(evt) {
      evt.preventDefault();
      this.$refs.downloadModal.hide();

      const path = "http://proxy.sptcalc.com/i94";
      //const path = "http://localhost:8000/i94";
      axios
        .post(path, this.downloadData)
        .then(res => {
          let history = res.data.travelHistory;
          if (history[0].eventType == "Arrival") {
            // Currently present, default trip until today
            this.trips.push({
              entry: history[0].eventDate,
              exit: moment().format("YYYY-MM-DD"),
              exempt: false
            });
            history.shift();
          }
          let lastEntry = NaN;
          let lastExit = NaN;
          for (let event of history) {
            if (event.eventType == "Arrival") {
              if (lastEntry)
                this.trips.push({
                  entry: lastEntry,
                  exit: lastExit,
                  exempt: false
                });
              lastEntry = event.eventDate;
            } else if (event.eventType == "Departure") {
              if (lastExit)
                this.trips.push({
                  entry: lastEntry,
                  exit: lastExit,
                  exempt: false
                });
              lastExit = event.eventDate;
            }
            if (lastEntry && lastExit) {
              this.trips.push({
                entry: lastEntry,
                exit: lastExit,
                exempt: false
              });
              lastEntry = NaN;
              lastExit = NaN;
            }
          }
        })
        .catch(error => {
          // eslint-disable-next-line
          console.log(error);
        });
    },
    daysPerYear: function(year) {
      return (
        moment(year, "YYYY")
          .endOf("year")
          .diff(moment(year, "YYYY").startOf("year"), "days") + 1
      );
    },
    isCompliantNonResident: function(year) {
      return this.compliantDays(year) >= 0;
    },
    compliantDays: function(year) {
      let currentYear = this.daysPresent(year);
      let remaining = Math.max(
        31 - currentYear,
        183 -
          (currentYear +
            this.daysPresent(year - 1) / 3 +
            this.daysPresent(year - 2) / 6)
      );
      return Math.floor(remaining) - 1;
    },
    daysPresent: function(year, excludePlanned = false) {
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
    onAddTrip(evt) {
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
    onCancelAdding(evt) {
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
    onEdit(evt) {
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
    onCancelEdit(evt) {
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