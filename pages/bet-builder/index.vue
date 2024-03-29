<template>
  <GeneralPage
    page_title="Bet Builder"
    class=""
    page_description="The Bet Builder page is an accumulation of fixtures with high prospects updated daily and powered by an ever learning algorithm that takes into account hundreds of high level data points exclusively available to Footy Amigo."
  >
    <template v-slot:howItWorks>
      <HowItWorks location="bet-builders" />
    </template>

    <div class="bet-builder-page">
      <!-- <div class="sort-lists d-flex justify-content-end">
        <sort-dropdown
          :sortList="sortList"
          :selected_sort="selected_sort"
          :title="'Sort By: '"
          @selected_sort="selected_sort"
        ></sort-dropdown>
      </div> -->
      <b-row no-gutters>
        <b-col cols="12">
          <hr />
        </b-col>
      </b-row>
      <b-row no-gutters class="mb-4">
        <FilteredButtons :markets="markets" v-model="market"> </FilteredButtons>
        <b-col cols="12" class="mt-3">
          <div class="footy-button-group" v-if="markets.length">
            <b-button
              class="bet-builder-type-button"
              :variant="type.value === selected_type ? 'primary' : 'light'"
              v-for="type in types"
              @click="selected_type = type.value"
              :key="'fixture-button' + type.value"
              :disabled="!$store.getters.subscriptionType"
            >
              {{ type.text }}
            </b-button>
          </div>
        </b-col>
      </b-row>

      <UpgradeToPro
        message="Upgrade to Pro to regain access to Bet Builder."
        :is_upgrade_overlay_shown="is_upgrade_overlay_shown"
      >
        <b-skeleton-wrapper :loading="loading">
          <template #loading>
            <div class="d-grid grid-col-md-2 gap-20">
              <div class="fixtures-wrapper">
                <div
                  class="fixture-container"
                  v-for="index in 2"
                  :key="'fixture_date_' + index"
                >
                  <div class="day">
                    <b-skeleton width="80px"></b-skeleton>
                  </div>
                  <div class="fixtures">
                    <div
                      class="
                          fixture
                          d-flex
                          align-items-center
                        "
                      v-for="k in 2"
                      :key="'fixture_' + k"
                    >
                      <b-skeleton width="100%" height="40px"></b-skeleton>
                    </div>
                  </div>
                </div>
              </div>
              <div class="rules-wrapper">
                <div class="rules-name">
                  <b-skeleton width="50%" height="30px"></b-skeleton>
                </div>
                <hr />
                <div class="rules-container">
                  <div
                    class="rule mb-1 d-flex align-items-center justify-content-between"
                    v-for="index in 4"
                    :key="'rule_' + index"
                  >
                    <b-skeleton width="100%" height="40px"></b-skeleton>
                  </div>
                </div>
              </div>
            </div>
          </template>
          <div class="d-grid grid-col-md-2 gap-20">
            <FixturesWrapper
              :marketFixtures="marketFixtures"
              :market="market"
              v-if="marketFixtures"
              :bets="bets"
              @showstats="showStats"
              @toggleBet="toggleBet"
            >
            </FixturesWrapper>

            <RulesWrapper
              v-if="selectedMarket"
              :selectedMarket="selectedMarket"
            >
            </RulesWrapper>
          </div>
        </b-skeleton-wrapper>
      </UpgradeToPro>
      <!--SlipButton
      :coins="coins"
      :betsLength="selectedBets.length"
      @toggleSelected="toggleSelected"
    ></SlipButton-->

      <SlipContent
        :coins="coins"
        :selectedBets="selectedBets"
        @clearBets="clearBets"
        @toggleBet="toggleBet"
        :fixtures="allFixtures"
      >
      </SlipContent>

      <ModalOnMobile
        v-model="show_fixture_details"
        v-if="show_fixture_details"
        :key="selected_fixture_id"
      >
        <FixtureDetails
          v-if="selected_fixture_id"
          @closestats="closeStats"
          :fixture_id="selected_fixture_id"
        >
        </FixtureDetails>
      </ModalOnMobile>
    </div>
  </GeneralPage>
</template>

<script>
import FilteredButtons from "~/components/betbuilder/FilteredButtons.vue";
import FixturesWrapper from "~/components/betbuilder/FixturesWrapper.vue";
import RulesWrapper from "~/components/betbuilder/RulesWrapper.vue";
//import SlipButton from "~/components/betbuilder/SlipButton.vue";
import SlipContent from "~/components/betbuilder/SlipContent.vue";
// import SortDropdown from "~/components/common/SortDropdown.vue";
import dummy_betbuilders from "~/components/json/dummy-betbuilders.json";

export default {
  name: "BetBuilderPage",
  head() {
    return {
      title: this.$titles.bet_builder,
    };
  },
  data() {
    return {
      sortList: ["League", "Time", "Home Win", "Away Win"],
      selected_sort: "League",
      bet_builder: {},
      market: "home_win",
      bets: {},
      selected_type: "all",
      is_upgrade_overlay_shown: false,
      types: [
        { value: "all", text: "All Fixtures" },
        { value: "last_5", text: "Last 5" },
        { value: "last_7", text: "Last 7" },
        { value: "last_10", text: "Last 10" },
        // { value: "last_25", text: "Last 25" }
      ],
      coins: 40,
      loading: true,
      selected_fixture_id: null,
      show_fixture_details: false,
    };
  },
  methods: {
    toggleSort(value) {
      this.selected_sort = value;
    },
    async fetchBets() {
      try {
        if (!this.$store.getters.subscriptionType) {
          return this.showDemoToExpired();
        }
        this.loading = true;
        const params = {
          type: this.selected_type,
        };

        this.bet_builder = await this.$axios.$get("/user/bet-builders", {
          params,
        });
        //this.market = Object.keys(this.bet_builder)[0];
      } catch (error) {
        console.log(error);
      } finally {
        this.loading = false;
      }
    },
    showDemoToExpired() {
      this.bet_builder = dummy_betbuilders;
      this.market = Object.keys(this.bet_builder)[0];
      this.is_upgrade_overlay_shown = true;
    },
    toggleBet(fixture_key) {
      const is_selected = this.bets[fixture_key];
      this.$set(this.bets, fixture_key, !is_selected);
    },
    clearBets() {
      this.bets = {};
    },
    clearBetBuilders() {
      this.bet_builder = {};
    },
    async showStats(fixture_id) {
      this.selected_fixture_id = fixture_id;
      this.show_fixture_details = true;
    },
    closeStats() {
      this.show_fixture_details = false;
    },
  },
  watch: {
    selected_type(val) {
      this.clearBets();
      //this.clearBetBuilders();
      this.fetchBets();
    },
  },
  computed: {
    selectedBets() {
      return Object.keys(this.bets).filter((bet) => this.bets[bet]);
    },
    allFixtures() {
      //Fixtures will store every fixtures in each bet market
      const fixtures = [];
      this.markets.map((market) => {
        const bet_builder = this.bet_builder[market.value];
        //Attach market name and key to each fixture in a market
        const bet_builder_fixtures = bet_builder.data.map((fixture) => {
          return {
            ...fixture,
            market_name: market.text,
            market_key: market.value,
          };
        });
        fixtures.push(...bet_builder_fixtures);
      });
      //Convert list of fixtures to an Object where fixture.key is the key of the object and fixture itself is the value
      return Object.assign(
        {},
        ...fixtures.map((fixture) => {
          return { [fixture.key]: fixture };
        })
      );
    },
    selectedMarket() {
      return this.bet_builder[this.market];
    },
    marketFixtures() {
      if (this.selectedMarket) {
        return this.selectedMarket.data;
      }
    },
    markets() {
      return Object.keys(this.bet_builder)
        .filter((key) => key != "total")
        .map((key) => {
          return {
            text: this.bet_builder[key].title,
            value: key,
          };
        });
    },
  },
  mounted() {
    this.fetchBets();
  },
  components: {
    FilteredButtons,
    FixturesWrapper,
    RulesWrapper,
    // SlipButton,
    SlipContent,
    // SortDropdown
  },
};
</script>

<style lang="scss">
.bet-builder-type-button {
  border-radius: 4px;
  padding: 5px 12px;
  font-size: 15px;
  line-height: 18px;
  font-weight: 500 !important;
}
</style>
