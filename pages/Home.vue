<template>
  <section id="home">
     <h1 hidden>Capitools DIY and gardening tools 1</h1>
    <SfHero
      class="hero desktop-only"
      :slider-options="{
        animationDuration: 2000,
        rewindDuration: 2000
      }"
    >
     <router-link to="gardening/chainsaws/new-premium-range-from-feider-machines-home-pro-petrol-chainsaws">
        <SfHeroItem
            class="sf-hero-item--position-bg-top-left sf-hero-item--align-right"
            image="/assets/banner/png/banner-03.png"
         />
      </router-link>
      <router-link to="/generators">
        <SfHeroItem
            class="sf-hero-item--position-bg-top-left sf-hero-item--align-right"
            image="/assets/banner/png/banner-01.png"
         />
      </router-link>
      <router-link to="/deals">
        <SfHeroItem
            class="sf-hero-item--position-bg-top-left sf-hero-item--align-right"
            image="/assets/banner/png/banner-02.png"
         />
      </router-link>
    </SfHero>
    <SfHero
      class="hero-tablet"
      :slider-options="{
        animationDuration: 2000,
        rewindDuration: 2000
      }"
    >
     <router-link to="gardening/chainsaws/new-premium-range-from-feider-machines-home-pro-petrol-chainsaws">
        <SfHeroItem
            class="sf-hero-item--position-bg-top-left sf-hero-item--align-right"
            image="/assets/banner/png/banner-03-responsive.png"
         />
      </router-link>
      <router-link to="/generators">
        <SfHeroItem
            class="sf-hero-item--position-bg-top-left sf-hero-item--align-right"
            image="/assets/banner/png/banner-01-responsive.png"
         />
      </router-link>
      <router-link to="/deals">
        <SfHeroItem
            class="sf-hero-item--position-bg-top-left sf-hero-item--align-right"
            image="/assets/banner/png/banner-02-responsive.png"
         />
      </router-link> 
    </SfHero>
     <MAssurance />   
    <OCmsBlockHomeTemplate /> 
    <div class="product-slider">
      <div class="container">    
        <div class="slider-content">  
          <m-product-carousel :products="ourBestDealsProducts" />
        </div> 
      </div>
    </div> 
    <OCmsBlockSwapText /> 
    <AAboutUs />
  </section>
</template>

<script>
import { mapState, mapGetters, mapActions } from 'vuex'; 
import { Logger } from '@vue-storefront/core/lib/logger';
import { isServer, onlineHelper } from '@vue-storefront/core/helpers'; 
import { checkWebpSupport } from 'theme/helpers'
import { ReviewModule } from '@vue-storefront/core/modules/review';
import { registerModule } from '@vue-storefront/core/lib/modules';
import { SfHero } from '@storefront-ui/vue';
import MProductCarousel from 'theme/components/molecules/m-product-carousel';
import MAssurance from 'theme/components/molecules/m-assurance';  
import OCmsBlockHomeTemplate from 'theme/components/organisms/o-cms-block-home-template';
import OCmsBlockSwapText from 'theme/components/organisms/o-cms-block-swap-text';
import AAboutUs from 'theme/components/atoms/a-about-us'; 
import { ModalList } from 'theme/store/ui/modals'

export default {
  name: 'Home',
  components: { 
    SfHero, 
    MProductCarousel,
    AAboutUs,
    MAssurance,    
    OCmsBlockHomeTemplate,
    OCmsBlockSwapText
  }, 
  data () {
    return {
      loading: true,
      loadNewsletterPopup: false
    };
  },
  computed: {
    ...mapState({
      isWebpSupported: state => state.ui.isWebpSupported,
      isFirstTimePopup: state => state.ui.isFirstTimePopup,
    }),
    ...mapGetters({
      isLoggedIn: 'user/isLoggedIn',   
      ourBestDealsProducts: 'homepage/getOurBestDealsProducts',
    }),
    isOnline () {
      return onlineHelper.isOnline;
    }
  }, 
  methods: {
    ...mapActions('ui', {
      openModal: 'openModal'
    }),
    handleScroll (event) {
      if(window.scrollY > 600){
        if(!localStorage.getItem('show_first_time_popup')){ 
          localStorage.setItem('show_first_time_popup',true);
          this.openModal({ name: ModalList.Subscribe });
        }
      }
    } 
  },
  watch: {
    isLoggedIn () {
      const redirectObj = localStorage.getItem('redirect');
      if (redirectObj) this.$router.push(redirectObj);
      localStorage.removeItem('redirect');
    }
  },
  async asyncData ({ store, context }) {
    Logger.info('Calling asyncData in Home (theme)')();
    if (context) context.output.cacheTags.add(`home`)

    await Promise.all([
      store.dispatch('homepage/ourBestDealsProducts')
    ]);
  },
   beforeCreate () {
    registerModule(ReviewModule);
  },
  mounted () {
    if (!this.isLoggedIn && localStorage.getItem('redirect')) { this.$bus.$emit('modal-show', 'modal-signup'); }
    window.addEventListener('scroll', this.handleScroll);
  },
  beforeDestroy () {
     window.removeEventListener('scroll', this.handleScroll);
  },
  beforeRouteEnter (to, from, next) {
    if (!isServer && !from.name) {
      next(vm => {
        vm.$store.dispatch('homepage/ourBestDealsProducts').then(() => {
           vm.loading = false;
       });
      });
    } else {
      next();
    }
  },
  metaInfo () {
    return {
      title: this.$route.meta.title || this.$i18n.t('Home Page'),
      meta: this.$route.meta.description ? [{ vmid: 'description', name: 'description', content: this.$route.meta.description }] : []
    }
  }
};
</script>

<style lang="scss" scoped>
@import "~@storefront-ui/shared/styles/helpers/breakpoints";

#home {
  box-sizing: border-box;
  padding: 0;
}

.sf-hero-item {
  --hero-item-height: 25rem;
  height: initial;
}
.banner-grid {
  margin: var(--spacer-base) 0;
  @include for-desktop {
    margin: var(--spacer-2xl) 0;
  }
}
.section {
  @include for-desktop {
    padding-left: 0;
    padding-right: 0;
  }
}
</style>