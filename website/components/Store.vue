<template>
  <div class="card mb-4 location-result">
    <a :id="`location-${store.properties.id}`" class="location-anchor" />
    <div class="card-header">
      <div class="row">
        <h5 class="col-sm mb-0">
          {{ store.properties.provider_brand_name }} -
          {{ store.properties.address }}, {{ store.properties.city }},
          {{ store.properties.state }}
          {{ store.properties.postal_code }}
        </h5>
        {{ /* Use v-show, not v-if for conditions without "else". Otherwise, strange things happen in production that cause rendering to fail (if the page is reloaded with a zip code pre-filled): https://github.com/nuxt/nuxt.js/issues/5800 */ }}
        <div v-show="store.distance" class="col-sm-auto">
          {{ store.distance }} miles
        </div>
      </div>
    </div>
    <div class="card-body">
      <div v-if="store.properties.appointments_available === true">
        <div class="location-status text-success fs-2">
          <font-awesome-icon icon="check-circle" class="align-middle" />
          <span class="fs-5"
            >Appointments available as of
            <display-local-time
              :time="appointmentsLastFetchedDate"
              :time-zone="store.properties.time_zone"
          /></span>
        </div>
        <appointment-times :store="store" />
        {{ /* Use v-show, not v-if for conditions without "else". Otherwise, strange things happen in production that cause rendering to fail (if the page is reloaded with a zip code pre-filled): https://github.com/nuxt/nuxt.js/issues/5800 */ }}
        <p v-show="store.properties.provider === 'kroger'" class="text-warning">
          <small
            ><font-awesome-icon icon="exclamation-triangle" />
            <strong>Warning:</strong> Many users are reporting issues booking
            appointments with {{ store.properties.provider_brand_name }} (due to
            2nd appointment requirements). However, some users have still
            reported success, so I still want to share the data I have from the
            pharmacies. I'm trying to figure out a better way to detect these
            issues, but in the meantime, sorry for any frustration!</small
          >
        </p>
        <p
          v-show="
            store.properties.provider === 'rite_aid' &&
            (store.properties.state === 'CA' ||
              store.properties.state === 'MI' ||
              store.properties.state === 'NY' ||
              store.properties.state === 'PA' ||
              store.properties.state === 'NJ' ||
              store.properties.state === 'OH' ||
              store.properties.state === 'OR' ||
              store.properties.state === 'WA')
          "
          class="text-warning"
        >
          <font-awesome-icon icon="exclamation-triangle" />
          <strong
            >Education Staff and Childcare Providers Only<span
              v-show="store.properties.state === 'NY'"
            >
              in NYC</span
            >:</strong
          >
          Rite Aid appointments are
          <a
            href="https://www.riteaid.com/corporate/news/-/pressreleases/news-room/2021/rite-aid-expands-and-prioritizes-covid-19-vaccine-eligibility-to-teachers-school-staff-and-childcare-providers"
            target="_blank"
            rel="noopener"
            >only bookable by teachers, school staff and childcare providers</a
          >
          until Saturday, March 13<span
            v-show="store.properties.state === 'NY'"
          >
            in NYC (outside of NYC other groups may still be eligible)</span
          >.
        </p>

        <a
          :href="store.properties.url"
          class="btn btn-primary"
          target="_blank"
          rel="noopener"
          >Visit {{ store.properties.provider_brand_name }} Website
          <font-awesome-icon icon="arrow-alt-circle-right"
        /></a>
      </div>
      <div v-else class="location-status">
        <div v-if="store.properties.appointments_available === false">
          <p class="text-danger">
            <font-awesome-icon icon="times-circle" />
            No appointments available as of last check
          </p>
        </div>
        <div v-else>
          <p>
            <font-awesome-icon icon="times-circle" />
            Unknown status
          </p>
          <p v-if="store.properties.carries_vaccine === false">
            At last check, this location does not carry the vaccine at all, so
            we have not fetched any appointments.
          </p>
          <p v-else-if="store.properties.appointments_last_fetched === null">
            We haven't collected any data for this pharmacy yet.
          </p>
          <p v-else>
            <strong>Uh oh!</strong> The data for this pharmacy is old. Please
            visit the
            <a :href="store.properties.url" target="_blank" rel="noopener"
              >pharmacy's website</a
            >
            directly for appointment availability. This likely means that the
            pharmacy is blocking our tool from accessing their site.
          </p>
        </div>
        <p>
          <a :href="store.properties.url" target="_blank" rel="noopener"
            >Visit {{ store.properties.provider_brand_name }} Website
            <font-awesome-icon icon="external-link-alt"
          /></a>
        </p>
      </div>

      <p class="card-text text-secondary mt-2">
        <small
          >Last checked
          <display-local-time
            v-if="store.properties.appointments_last_fetched"
            :time="appointmentsLastFetchedDate"
          />
          <span v-if="!store.properties.appointments_last_fetched"
            >never</span
          ></small
        >
      </p>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    store: {
      type: Object,
      required: true,
    },
  },

  data() {
    return {
      loaded: false,
    };
  },

  computed: {
    appointmentsLastFetchedDate() {
      return new Date(this.store.properties.appointments_last_fetched);
    },
  },

  created() {
    // "loaded" workaround due to odd issues in built production mode (if zip
    // code is set and the page is reloaded then things fail to render):
    // https://github.com/nuxt/nuxt.js/issues/5800#issuecomment-613739824
    this.$nextTick(() => {
      this.loaded = true;
    });
  },
};
</script>

<style>
.location-result .location-status {
  line-height: 1rem;
  margin-bottom: 1rem;
}

.text-warning {
  color: #d99011 !important;
}

.location-anchor {
  padding-top: 56px;
  margin-top: -56px;
}
</style>
