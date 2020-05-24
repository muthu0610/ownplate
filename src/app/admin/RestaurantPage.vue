<template>
  <section class="section">
    <back-button url="/admin/restaurants/" />

    <template v-if="notFound===null"></template>
    <template v-else-if="notFound">
      <not-found />
    </template>
    <template v-else-if="!notFound">
      <h2 class="p-big bold">{{$t("editRestaurant.about")}}</h2>
      <div class="media">
        <div class="media-content"></div>
        <div class="media-right">
          <p class="p-font bold" style="color:#CB4B4B">* {{$t("editRestaurant.required")}}</p>
        </div>
      </div>

      <div class="field is-horizontal">
        <div class="field-body">
          <h4>
            {{$t("editRestaurant.profilePhoto")}}
            <span class="p-font bold" style="color:#CB4B4B">*</span>
          </h4>
        </div>
      </div>
      <table>
        <tr>
          <td v-if="restProfilePhoto">{{$t("editCommon.current")}}</td>
          <td>{{$t("editCommon.new")}}</td>
        </tr>
        <tr>
          <td v-if="restProfilePhoto">
            <img class="card_image" :src="restProfilePhoto" />
          </td>
          <td>
            <croppa
              :prevent-white-space="true"
              :zoom-speed="5"
              :width="200"
              :height="200"
              :accept="'image/jpeg'"
              :placeholder="$t('editCommon.clickAndUpload')"
              :placeholder-font-size="10"
              :disable-drag-to-move="true"
              :disable-scroll-to-zoom="true"
              :disable-rotation="true"
              initial-position="center"
              :canvas-color="'gainsboro'"
              @file-choose="handleProfileImage"
            ></croppa>
          </td>
        </tr>
      </table>
      <div class="field is-horizontal">
        <div class="field-body">
          <h4>{{$t("editRestaurant.coverPhoto")}}</h4>
        </div>
      </div>
      <table>
        <tr>
          <td v-if="restCoverPhoto">{{$t("editCommon.current")}}</td>
          <td>{{$t("editCommon.new")}}</td>
        </tr>
        <tr>
          <td v-if="restCoverPhoto">
            <img
              class="card_cover_image"
              :src="restCoverPhoto"
            />
          </td>
          <td>
            <croppa
              :prevent-white-space="true"
              :zoom-speed="5"
              :width="200"
              :height="200"
              :accept="'image/jpeg'"
              :placeholder="$t('editCommon.clickAndUpload')"
              :placeholder-font-size="10"
              :disable-drag-to-move="true"
              :disable-scroll-to-zoom="true"
              :disable-rotation="true"
              initial-position="center"
              :canvas-color="'gainsboro'"
              @file-choose="handleCoverImage"
            ></croppa>
          </td>
        </tr>
      </table>
      <text-form
        v-model="shopInfo.restaurantName"
        titleKey='shopInfo.name'
        placeHolder='editRestaurant.enterRestaurantName'
        :error="errors['restaurantName']"
        :maxlength='50' />

      <template v-if="region==='JP'">
        <text-form :error="errors['zip']"
                   v-model="shopInfo.zip"
                   titleKey='shopInfo.zip'
                   placeHolder='editRestaurant.enterZip'
                   :maxlength='10' />
        <div class="columns">
          <state :errors="errors" v-model="shopInfo.state" />
          <div class="column">
            <text-form
              :error="errors['city']"
              v-model="shopInfo.city"
              titleKey='shopInfo.city'
              placeHolder='editRestaurant.enterCity'
              :maxlength='15' />
          </div>
        </div>
        <text-form
          :error="errors['streetAddress']"
          v-model="shopInfo.streetAddress"
          titleKey='shopInfo.streetAddress'
          placeHolder='editRestaurant.enterStreetAddress'
          :maxlength='30' />
      </template>

      <template v-else>
        <text-form
          :error="errors['streetAddress']"
          v-model="shopInfo.streetAddress"
          titleKey='shopInfo.streetAddress'
          placeHolder='editRestaurant.enterStreetAddress'
          :maxlength='30' />

        <div class="columns">
          <text-form :error="errors['city']"
                     v-model="shopInfo.city"
                     titleKey='shopInfo.city'
                     placeHolder='editRestaurant.enterCity'
                     :maxlength='15' />
          <div class="column">
            <state :errors="errors" v-model="shopInfo.state" />
          </div>
        </div>
        <text-form :error="errors['zip']"
                   v-model="shopInfo.zip"
                   titleKey='shopInfo.zip'
                   placeHolder='editRestaurant.enterZip'
                   :maxlength='10' />
      </template>


      <b-field>
        <b-button
          variant="outline-primary"
          style="margin-right:auto"
          type="is-primary"
          class="counter-button"
          rounded
          @click="updateAndUpdateMap"
        >{{$t("editRestaurant.updateMap")}}</b-button>
        <br />
      </b-field>
      <div class="field is-horizontal">
        <div class="field-body">
          <p class="p-font bold" style="color:#CB4B4B">{{$t('editRestaurant.updateMapDescription')}}</p>
        </div>
      </div>
      <b-field type="is-white">
        <GMap
          ref="gMap"
          :center="{lat: 44.933076, lng: 15.629058}"
          :options="{fullscreenControl: false}"
          :zoom="18"
          @loaded="hello"
        ></GMap>
      </b-field>

      <div class="field is-horizontal">
        <div class="field-body">
          <h4>
            {{$t('shopInfo.phonenumber')}}
            <span class="p-font bold" style="color:#CB4B4B">*</span>
          </h4>
        </div>
      </div>
      <phone-entry :currentNumber="shopInfo.phoneNumber"
                  :placeHolder="$t('editRestaurant.enterPhone')" @change="handlePhoneChange"/>

      <text-form v-model="shopInfo.url"
                 :error="errors['url']"
                 titleKey='shopInfo.website'
                 placeHolder='editRestaurant.enterWebsite'
                 :maxlength='100'
                 :required="false" />

      <text-form v-model="shopInfo.introduction"
                 type="textarea"
                 :required="false"
                 :maxlength='300'
                 titleKey='editRestaurant.introduction'
                 placeHolder='editRestaurant.enterIntroduction'
                 :error="errors['introduction']"
                 />

      <text-form v-model="shopInfo.orderNotice"
                 type="textarea"
                 :required="false"
                 :maxlength='300'
                 titleKey='editRestaurant.orderNotice'
                 placeHolder='editRestaurant.enterOrderNotice'
                 :error="errors['orderNotice']"
                 />

      <text-form v-model="shopInfo.orderThanks"
                 type="textarea"
                 :required="false"
                 :maxlength='300'
                 titleKey='editRestaurant.orderThanks'
                 placeHolder='editRestaurant.enterOrderThanks'
                 :error="errors['orderThanks']"
                 />


      <div class="columns" v-if="requireTaxInput">
        <div class="column">
          <div class="field is-horizontal">
            <div class="field-body">
              <div class="field has-addons">
                <b-field
                  :label="$t('editRestaurant.foodTax')"
                  style="border-radius: 0.4rem!important;"
                  :type="errors['foodTax'].length > 0 ? 'is-danger' : 'is-success'"
                >
                  <div style="display:inline-flex">
                    <b-input
                      v-model="shopInfo.foodTax"
                      placeholder="8.2"
                      type="text"
                      maxlength="5"
                    />
                    <span
                      style="margin-top: auto;margin-bottom: auto;margin-left:0.4rem;margin-right:0.4rem;"
                    >%</span>
                  </div>
                </b-field>
              </div>
            </div>
          </div>
        </div>
        <div class="column">
          <div class="field is-horizontal">
            <div class="field-body">
              <div class="field has-addons">
                <b-field
                  :label="$t('editRestaurant.alcoholTax')"
                  style="border-radius: 0.4rem!important;"
                  :type="errors['alcoholTax'].length > 0 ? 'is-danger' : 'is-success'"
                >
                  <div style="display:inline-flex">
                    <b-input
                      v-model="shopInfo.alcoholTax"
                      placeholder="10.2"
                      type="text"
                      maxlength="5"
                    />
                    <span
                      style="margin-top: auto;margin-bottom: auto;margin-left:0.4rem;margin-right:0.4rem;"
                    >%</span>
                  </div>
                </b-field>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="field is-horizontal" v-if="!requireTaxInput">
        <div class="field-body">
          <h4>
            {{$t('editRestaurant.tax')}}
          </h4>
        </div>
      </div>
      <div v-if="!requireTaxInput" >
        <span v-for="taxItem in taxRates">
          {{$t('editMenu.' + taxRateKeys[taxItem])}} {{shopInfo[taxItem + "Tax"]}}% <br/>
        </span>
        <br/>
     </div>
      <div class="field is-horizontal" v-if="requireTaxPriceDisplay">
        <div class="field-body">
          <h4>
            {{$t('editRestaurant.taxPriceDisplay')}}
          </h4>
        </div>
      </div>

      <div v-if="requireTaxPriceDisplay">
        <span>
          <b-checkbox v-model="shopInfo.taxInclude">{{$t("editRestaurant.taxIncluded")}}</b-checkbox>
        </span>
        <br/>
        {{$tc('tax.taxExample', examplePriceI18n)}} - <Price :shopInfo="shopInfo" :menu="sampleMenu"/></span>
        <br/>
      </div>

      <h4>{{$t('shopInfo.hours')}}</h4>
      <p class="p-font bold" style="color:#CB4B4B">{{$t('editRestaurant.businessHourDescription')}}</p>
      <div v-for="(day, index) in days" :key="index">
        <div class="field">
          <b-checkbox v-model="shopInfo.businessDay[index]">{{$t("week.short." + day)}}</b-checkbox>
        </div>
        <div class="hours">
          <hours-input
            v-model="shopInfo.openTimes[index][0]"
            :type="errors['time'][index][0].length > 0 ? 'is-danger' : 'is-success'"
            :disabled="!shopInfo.businessDay[index]"
            ></hours-input>
          <hours-input
            v-model="shopInfo.openTimes[index][1]"
            :type="errors['time'][index][1].length > 0 ? 'is-danger' : 'is-success'"
          :disabled="!shopInfo.businessDay[index]"
            ></hours-input>
        </div>
      </div>

      <b-checkbox v-model="shopInfo.publicFlag"
                  :disabled="hasError"
                  :type="!shopInfo.publicFlag ? 'is-danger' : ''">
        {{$t('shopInfo.public')}}
      </b-checkbox><br/>

      <span v-if="shopInfo.publicFlag">{{$t('editRestaurant.publishDescription')}}<br/></span>
      <span style="color:#CB4B4B" v-if="!shopInfo.publicFlag">{{$t('editRestaurant.draftDescription')}}<br/></span>
      <span v-if="hasError" class="p-font bold" style="color:#CB4B4B">{{$t('editRestaurant.draftWarning')}}<br/></span>
      <b-button
        style="margin-right:auto"
        type="is-primary"
        class="counter-button save_btn"
        :disabled="submitting"
        rounded
        @click="submitRestaurant"
      >{{$t(submitting ? 'editCommon.saving' : (shopInfo.publicFlag ? 'editCommon.save' : 'editCommon.saveDraft') )}}</b-button>
    </template>
  </section>
</template>

<script>
import Vue from "vue";
import { db, storage, firestore } from "~/plugins/firebase.js";

import * as API from "~/plugins/api";
import BackButton from "~/components/BackButton";
import NotFound from "~/components/NotFound";
import PhoneEntry from "~/components/PhoneEntry";
import Price from "~/components/Price";
import { ownPlateConfig } from "@/config/project";


import HoursInput from "./inputComponents/HoursInput";
import TextForm from "./inputComponents/TextForm"
import State from "./inputComponents/State"

import { taxRates, daysOfWeek, regionalSettings } from "~/plugins/constant.js";

export default {
  name: "Order",
  components: {
    HoursInput,
    TextForm,
    State,
    BackButton,
    NotFound,
    PhoneEntry,
    Price
  },

  data() {
    const regionalSetting = regionalSettings[ownPlateConfig.region || "US"];

    return {
      taxRates: taxRates,
      taxRateKeys: regionalSetting["taxRateKeys"],

      examplePriceI18n: this.$n(1000, 'currency'),
      sampleMenu: {price: 1000, tax: 'food'},
      requireTaxInput: regionalSetting.requireTaxInput,
      requireTaxPriceDisplay: regionalSetting.requireTaxPriceDisplay,

      defaultTax: regionalSetting.defaultTax,
      disabled: false, // ??
      filteredItems: [], // ??
      test: null,
      shopInfo: {
        restaurantName: "",
        streetAddress: "",
        city: "",
        state: "",
        zip: "",
        location: {},
        place_id: null,
        phoneNumber: "",
        url: "",
        introduction: "",
        orderNotice: "",
        orderThanks: "",
        foodTax: 0,
        alcoholTax: 0,
        taxInclude: 0,
        openTimes: {
          1: [], // mon
          2: [],
          3: [],
          4: [],
          5: [],
          6: [],
          7: []
        },
        businessDay: {
          1: true, // mon
          2: true,
          3: true,
          4: true,
          5: true,
          6: true,
          7: true
        },
        images: {},
        publicFlag: false
      },
      region: ownPlateConfig.region,
      maplocation: {},
      place_id: null,
      markers: [],
      days: daysOfWeek,
      errorsPhone: [],
      notFound: null,
      submitting: false,
      files: {},
    };
  },
  async created() {
    this.checkAdminPermission();

    // never use onSnapshot here.
    const restaurant = await db.doc(`restaurants/${this.restaurantId()}`).get();

    if (!restaurant.exists) {
      this.notFound = true;
      return;
    }
    const restaurant_data = restaurant.data();
    if (restaurant_data.uid !== this.uid) {
      this.notFound = true;
      return;
    }
    this.shopInfo = Object.assign({}, this.shopInfo, restaurant_data);
    if (this.defaultTax) {
      this.shopInfo = Object.assign({}, this.shopInfo, this.defaultTax);
    }
    this.notFound = false;
  },
  mounted() {
    this.hello();
  },
  computed: {
    restProfilePhoto() {
      return (this.shopInfo?.images?.profile?.resizedImages || {})["600"] || this.shopInfo.restProfilePhoto;
    },
    restCoverPhoto() {
      return (this.shopInfo?.images?.cover?.resizedImages || {})["600"] || this.shopInfo.restCoverPhoto;
    },
    uid() {
      return this.$store.getters.uidAdmin;
    },
    errors() {
      const err = {};
      [
        "restaurantName",
        "streetAddress",
        "city",
        "state",
        "zip",
        "phoneNumber",
      ].forEach(name => {
        err[name] = [];
        if (this.shopInfo[name] === "") {
          err[name].push("validationError." + name + ".empty");
        }
      });
      ["introduction", "orderNotice", "orderThanks"].forEach(name => {
        err[name] = [];
      });
      if (this.requireTaxInput) {
        ["foodTax", "alcoholTax"].forEach(name => {
          err[name] = [];
          if (this.shopInfo[name] === "") {
            err[name].push("validationError." + name + ".empty");
          }
          if (this.shopInfo[name] !== "") {
            if (isNaN(this.shopInfo[name])) {
              err[name].push("validationError." + name + ".invalidNumber");
            }
          }
        });
      }

      const ex = new RegExp("^(https?)://[^\\s]+$");
      err["url"] =
        this.shopInfo.url && !ex.test(this.shopInfo.url)
          ? ["validationError.url.invalidUrl"]
          : [];

      err["time"] = {};
      Object.keys(daysOfWeek).forEach(key => {
        err["time"][key] = [];
        [0, 1].forEach(key2 => {
          err["time"][key].push([]);
          if (this.shopInfo.businessDay[key]) {
            if (
              this.shopInfo.openTimes[key] &&
              this.shopInfo.openTimes[key][key2]
            ) {
              const data = this.shopInfo.openTimes[key][key2];
              if (this.isNull(data.start) ^ this.isNull(data.end)) {
                err["time"][key][key2].push("validationError.oneInEmpty");
              }
              if (!this.isNull(data.start) && !this.isNull(data.end)) {
                if (data.start > data.end) {
                  err["time"][key][key2].push(
                    "validationError.validBusinessTime"
                  );
                }
              }
            } else {
              if (key2 === 0) {
                err["time"][key][key2].push("validationError.noSelect");
              }
            }
          }
        });
      });
      err["phoneNumber"] = this.errorsPhone;
      // todo more validate
      return err;
    },
    hasError() {
      const num = this.countObj(this.errors);
      return num > 0;
    }
  },
  watch: {
    notFound: function() {
      if (this.notFound === false) {
        this.hello();
      }
    },
    hasError: function() {
      this.shopInfo.publicFlag = !this.hasError;
    },
    files: function() {
      console.log(this.files);
    }
  },
  methods: {
    handleProfileImage(e) {
      this.files["profile"] = e;
    },
    handleCoverImage(e) {
      this.files["cover"] = e;
    },
    handlePhoneChange(payload) {
      //console.log(payload)
      this.shopInfo.phoneNumber = payload.phoneNumber;
      this.shopInfo.countryCode = payload.countryCode;
      this.errorsPhone = payload.errors;
    },
    hello() {
      if (this.shopInfo && this.shopInfo.location) {
        this.setCurrentLocation(this.shopInfo.location);
      }
    },
    async submitRestaurant() {
      this.submitting = true;
      const restaurantId = this.restaurantId();
      try {
        if (this.files["profile"]) {
          const path = `/images/restaurants/${restaurantId}/${this.uid}/profile.jpg`;
          this.shopInfo.restProfilePhoto = await this.uploadFile(this.files["profile"], path);
          this.shopInfo.images.profile = {
            original: this.shopInfo.restProfilePhoto,
            resizedImages: {},
          };
        }

        if (this.files["cover"]) {
          const path = `/images/restaurants/${restaurantId}/${this.uid}/cover.jpg`;
          this.shopInfo.restCoverPhoto = await this.uploadFile(this.files["cover"], path);
          this.shopInfo.images.cover = {
            original: this.shopInfo.restCoverPhoto,
            resizedImages: {},
          };
        }
        const restaurantData = {
          restProfilePhoto: this.shopInfo.restProfilePhoto,
          restCoverPhoto: this.shopInfo.restCoverPhoto,
          restaurantName: this.shopInfo.restaurantName,
          streetAddress: this.shopInfo.streetAddress,
          images: {
            cover: this.shopInfo?.images?.cover || {} ,
            profile: this.shopInfo?.images?.profile || {} ,
          },
          city: this.shopInfo.city,
          state: this.shopInfo.state,
          zip: this.shopInfo.zip,
          location: this.shopInfo.location,
          place_id: this.shopInfo.place_id,
          phoneNumber: this.shopInfo.phoneNumber,
          countryCode: this.shopInfo.countryCode,
          url: this.shopInfo.url,
          introduction: this.shopInfo.introduction,
          orderNotice: this.shopInfo.orderNotice,
          orderThanks: this.shopInfo.orderThanks,
          foodTax: Number(this.shopInfo.foodTax),
          alcoholTax: Number(this.shopInfo.alcoholTax),
          openTimes: this.shopInfo.openTimes,
          businessDay: this.shopInfo.businessDay,
          uid: this.shopInfo.uid,
          publicFlag: this.shopInfo.publicFlag,
          taxInclude: this.shopInfo.taxInclude,
          createdAt: this.shopInfo.createdAt || firestore.FieldValue.serverTimestamp(),
        };
        await this.updateRestaurantData(restaurantData);

        this.$router.push({
          path: `/admin/restaurants/`
        });
      } catch (error) {
        this.submitting = false;
        this.$store.commit("setErrorMessage", {
          code: "restaurant.save",
          error
        });
      }
    },
    async updateAndUpdateMap() {
      await this.updateMap();
      this.setLocation();
    },
    async updateMap() {
      // https://gitlab.com/broj42/nuxt-gmaps#readme
      // https://codesandbox.io/s/6j6zw48l83
      const keyword = [
        this.shopInfo.restaurantName,
        this.shopInfo.streetAddress,
        this.shopInfo.city,
        this.shopInfo.state
      ].join(",");

      const res = await API.google_geocode(keyword);
      if (res && res[0] && res[0].geometry) {
        this.setCurrentLocation(res[0].geometry.location);
        this.place_id = res[0].place_id;
      }
    },
    setCurrentLocation(location) {
      if (
        this.$refs.gMap &&
        this.$refs.gMap.map &&
        location &&
        location.lat &&
        location.lng
      ) {
        this.$refs.gMap.map.setCenter(location);
        this.removeAllMarker();
        const marker = new google.maps.Marker({
          position: new google.maps.LatLng(location.lat, location.lng),
          title: "hello",
          map: this.$refs.gMap.map
        });
        this.markers.push(marker);
        this.maplocation = location;
      }
    },
    setLocation() {
      if (this.maplocation) {
        this.shopInfo.location = this.maplocation;
        this.shopInfo.place_id = this.place_id;
      }
    },
    removeAllMarker() {
      if (this.markers && this.markers.length > 0) {
        this.markers.map(marker => {
          marker.setMap(null);
        });
        this.markers = [];
      }
    },
    async updateRestaurantData(restaurantData) {
      const cleanData = this.cleanObject(restaurantData);
      await db.doc(`restaurants/${this.restaurantId()}`).update(cleanData);
    }
  }
};
</script>
<style lang="scss" scoped>
.hours {
    margin-bottom: 0.75rem;
}
.tax {
  margin-top: -2rem !important;
}
.card_image {
    height: 200px;
    max-width: 200px;
}
.card_cover_image {
    height: 200px;
    max-width: 200px;
}
.save_btn {
  position: fixed;
  bottom: 2rem;
  width: 80%;
  left: 50%;
  transform: translate(-50%, 0);
  margin-left: auto;
  margin-right: auto;
  height: 4rem;
  font-size: 1.5rem;
  font-weight: 200;
  z-index: 10;
}
/deep/.ti-input {
  border-radius: 0.4rem !important;
}
</style>