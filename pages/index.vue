<template>
  <div class="vh-100 min-vw-100 bg-light">
    <div class="container-fluid h-100">
      <div class="row h-100">
        <div class="col-12 col-sm-6 col-lg-4 col-xl-5 h-100 pe-lg-0">
          <div class="w-100 pe-4">
            <select
              id="province_filter"
              class="form-select my-2 me-5"
              @change="filterProvinces"
            >
              <option>All Provinces</option>
              <option>Alberta</option>
              <option>British Columbia</option>
              <option>Ontario</option>
            </select>
          </div>

          <div id="member-list">
            <div
              v-for="province in provinces"
              :key="provinces.indexOf(province)"
            >
              <div class="py-2 text-center bg-danger text-white fw-bold">
                {{ province }}
              </div>

              <div v-for="member in locations" :key="member.id">
                <div
                  :class="'card mb-2 ms-1 me-2 ' + member.selected"
                  @click="goToMarker(member)"
                  :data-member-id="member.id"
                  v-if="member.visible && member.Province === province"
                >
                  <div class="row">
                    <div
                      class="
                        col-md-5 col-12
                        text-center
                        d-flex
                        justify-content-center
                        align-items-lg-center
                      "
                    >
                      <img
                        :src="member.Logo"
                        class="img-fluid mx-4 memberLogo"
                        alt="#"
                      />
                    </div>

                    <div class="col-md-7 col-12 text-center text-md-start px-2">
                      <div class="card-body ps-0">
                        <div class="card-title">
                          <div
                            class="
                              d-flex
                              mb-2
                              justify-content-center justify-content-md-start
                            "
                          >
                            <img
                              src="https://emrg.com/wp-content/uploads/2019/12/logo.png"
                              :alt="member.Title"
                              :title="member.Title"
                              class="me-2 mt-1"
                              style="max-height: 1.25rem; object-fit: contain"
                            />
                            <h2 class="h5 fw-bold mb-0">
                              {{ member.Title }}
                            </h2>
                          </div>
                        </div>
                        <div class="d-flex">
                          <div class="col-md-7 col-12">
                            <p
                              class="card-text small"
                              v-html="
                                '<strong>' +
                                member.Member +
                                '</strong><br>' +
                                member.Address
                              "
                            ></p>
                          </div>
                          <div class="col-md-5 col-12">
                            <div class="card-text small text-muted d-block">
                              <i class="fa fa-phone fa-fw me-2"></i>
                              <a href="tel:18443443674">Call</a>
                              <br />
                              <i class="fa fa-envelope fa-fw me-2"></i>
                              <a :href="'mailto:' + member.Email">Email</a>
                            </div>
                          </div>
                        </div>

                        <!-- <i class="fa fa-globe me-2"></i>
                      <a :href="member.url" target="_blank"> Website</a> -->
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
        <div id="map-wrap" class="col h-100 ps-lg-0">
          <client-only>
            <l-map
              ref="myMap"
              @ready="initMap"
              :zoom="7"
              :center="[50.9464418, -114.01277591]"
              v-if="!isMobile()"
            >
              <l-tile-layer
                url="https://stamen-tiles-{s}.a.ssl.fastly.net/toner-lite/{z}/{x}/{y}{r}.png"
                attribution="Built by <a href='https://barrellmarketing.com'>Barrel Marketing</a>"
              ></l-tile-layer>
              <l-layer-group ref="members">
                <l-marker
                  v-for="member in locations"
                  :key="member.id"
                  :lat-lng="member.latlong"
                  @click="goToMarker(member)"
                  v-show="member.visible"
                  ><l-icon
                    iconUrl="favicon.png"
                    :icon-size="[54, 54]"
                    :className="isSelected(member.selected)"
                  ></l-icon
                ></l-marker>
              </l-layer-group>
            </l-map>
          </client-only>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import locations from "@/assets/locations.json";
export default {
  name: "IndexPage",
  data() {
    return {
      locations: locations,
      map: null,
      provinces: [],
    };
  },

  mounted() {
    var provinces = [];
    for (var i = 0; i < this.locations.length; i++) {
      // console.log(this.locations[i].Province);
      if (!provinces.includes(this.locations[i].Province)) {
        provinces.push(locations[i].Province);
      }
    }
    this.provinces = provinces;
    console.log(this.provinces);
  },
  computed: {
    selectedMember() {
      return this.locations.find((member) => member.selected);
    },
    getLocations() {
      function compare(el1, el2, index) {
        return el1[index] == el2[index] ? 0 : el1[index] < el2[index] ? -1 : 1;
      }
      return this.locations.sort(function (el1, el2) {
        var compared = compare(el1, el2, "Province");
        return compared == 0 ? -compare(el1, el2, "Member Name") : compared;
      });
    },
  },
  methods: {
    initMap() {
      console.log(this.$refs.members);
      this.map = this.$refs.myMap.mapObject;
      this.map.fitBounds(
        this.locations.map((m) => {
          return m.latlong;
        })
      );
    },
    openPopUp(latLng, caller) {
      this.caller = caller;
      // this.$refs.members.mapObject.openPopup(latLng);
    },
    goToMarker(member) {
      if (this.isMobile()) {
        return;
      }
      this.map.setView(member.latlong, 7);

      for (var i in this.locations) {
        if (this.locations[i].id !== member.id) {
          console.log(this.locations[i]);
          this.locations[i].selected = false;
        }
      }
      member.selected = "selected";
      document
        .querySelector('[data-member-id="' + member.id + '"]')
        .scrollIntoView({
          behavior: "auto",
          block: "center",
          inline: "center",
        });
    },
    isSelected(selected) {
      if (selected) {
        return "selected";
      } else {
        return "";
      }
    },
    filterProvinces(e) {
      console.log(e.target.value);
      if (e.target.value === "All Provinces") {
        this.locations.forEach(function (member) {
          member.visible = true;
        });
      } else {
        this.locations.forEach(function (member) {
          if (member.Province === e.target.value) {
            member.visible = true;
          } else {
            member.visible = false;
          }
        });
      }
      if (!this.isMobile()) {
        this.map = this.$refs.myMap.mapObject;
        this.map.fitBounds(
          this.locations.map((m) => {
            if (m.visible) return m.latlong;
          })
        );
      }
    },
    isMobile() {
      if (!process.client) {
        return;
      }
      if (
        /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(
          window.navigator.userAgent
        )
      ) {
        console.log("is mobile");
        return true;
      } else {
        return false;
      }
    },
  },
};
</script>

<style>
</style>
