<template>
  <div>
    <div class="white-box mb-3">
      <div class="d-flex justify-content-between">
        <div class="content">
          <label class="footy-label">Make Public</label>
          <h6 class="description-text">
            Making your strategy public will allow it to be found by other Footy
            Amigo Pro users and also allows you to be able to see profitable
            strategies from other users.
          </h6>
        </div>
        <div class="btn">
          <footy-switch v-model="form.is_public" name="make-public">
          </footy-switch>
        </div>
      </div>
    </div>
    <div>
      <label class="footy-label">When Should the Alert Be Sent?</label>
      <h6 class="description-text" style="margin-bottom: 12px">
        F.e. 15 mins before match starts
      </h6>

      <b-form-radio-group
        class="footy-radio-group mb-4 wrap-on-mobile"
        v-model="form.timer.minute"
        :options="timer"
        button-variant="light"
        name="timer"
        buttons
      ></b-form-radio-group>
    </div>
    <b-overlay style="min-height: 300px" :show="loading">
      <footy-vertical-checkbox
        v-if="!loading"
        v-model="form.leagues"
        :options="$store.state.leagues"
        :value="globalIncludedLeagues"
        name="leagues"
        id="leagues"
        class="mb-4"
        label="Select Leagues"
        useGlobalLabel="Use Global Leagues"
        :globalValue="globalIncludedLeagues"
        filtered
        outcomes
      ></footy-vertical-checkbox>
    </b-overlay>

    <div class="d-flex my-3">
      <h4 class="mr-2">Add Note</h4>
      <footy-switch v-model="form.has_note" name="add-note"> </footy-switch>
    </div>
    <b-textarea
      v-show="form.has_note"
      v-model="form.note"
      style="border-radius: 8px; transition: visibility 0s, opacity 0.5s linear"
    >
    </b-textarea>
    <div class="step-button-group mt-5">
      <b-button
        class="footy-button"
        @click="$emit('prevstep')"
        href="#create-strategy"
      >
        <PrevIcon class="icon-left" />
        Previous Step
      </b-button>

      <b-button
        class="footy-button"
        variant="primary"
        :disabled="form.leagues.length == 0"
        @click="$emit('save')"
        href="#create-strategy"
      >
        Finish & Save
        <NextIcon class="icon-right" />
      </b-button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    value: Object,
  },
  // scrollToTop: true,
  data() {
    return {
      selected: 30,
      globalIncludedLeagues: [],
      loading: true,
      timer: [
        {value: 15, text: '15 mins before'},
        {value: 30, text: '30 mins before'},
        {value: 60, text: '1 hour before'},
      ],
    };
  },
  mounted() {
    this.fetchGlobalIncludedLeagues();
  },
  methods: {
    async fetchGlobalIncludedLeagues() {
      // console.log('@smart: fetchGlobalIncludedLeagues');
      this.loading = true;
      const params = {strategy_type: 'pre-match-alerts'};
      const globalIncludedLeagues = await this.$axios.$get(
        `/user/leagues/league-all/`,
        {
          params,
        }
      );
      this.globalIncludedLeagues = globalIncludedLeagues.map(({id}) => id);
      // console.log('@smart: fetchGlobalIncludedLeagues', this.globalIncludedLeagues);
      this.loading = false;
    },
  },
  computed: {
    form: {
      get() {
        return this.value;
      },
      set(val) {
        this.$emit('input', val);
      },
    },
  },
};
</script>

<style lang="scss">
@import '~/assets/scss/colors';
.white-box {
  background: #ffffff;
  border: 1px solid #d5ded5;
  box-sizing: border-box;
  border-radius: 12px;
  padding: 16px;
}

.flex-justify-space-between {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.center {
  display: flex;
  align-items: center;
  vertical-align: middle;
}

.flex-1 {
  flex: 1;
}
</style>
