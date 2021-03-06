<template>
  <div class="m-product-short-info">
    <div class="product__header">
      <SfHeading
        :title="product.name | htmlDecode"
        :level="3"
        class="sf-heading--no-underline sf-heading--left"
      />
      <SfIcon
        icon="drag"
        size="xl"
        color="gray-secondary"
        class="product__drag-icon mobile-only"
      />
    </div>
    <p class="reference">
      <b>Reference : </b> {{ product.sku }}
    </p>
    <div class="product__price-and-rating">
      <AProductRating
        @click="openReviewsTab"
        :reviews="reviews"
        class="d-flex"
      >
      </AProductRating>
    </div>
    <hr>
    <div class="product__description desktop-only" v-html="this.productDescription" />
    <a @click="openDescriptionTab" class="read-more">Read more</a>
    <div v-if="showOrHideBlocksBasedOnBFTG(product)">
      <AMesAdvantage />
    </div>
    <div v-else>
      <AGarantie />
    </div>
  </div>
</template>

<script>
import config from 'config';
import { SfHeading, SfIcon, SfPrice, SfButton } from '@storefront-ui/vue';
import AProductRating from 'theme/components/atoms/a-product-rating';
import AProductPrice from 'theme/components/atoms/a-product-price';
import AMesAdvantage from 'theme/components/atoms/a-mes-advantage';
import AGarantie from 'theme/components/atoms/a-garantie';
import { createSmoothscroll } from 'theme/helpers'
export default {
  name: 'MProductShortInfo',
  components: {
    SfHeading,
    SfIcon,
    AProductRating,
    AProductPrice,
    AMesAdvantage,
    AGarantie,
    SfButton
  },
    data() {
      return { 
          productDescription:'',
          imagemediapath:'{{media url="wysiwyg',
          imagepng:'.png"}}',
          imagejpg:'.jpg"}}',
          imagejpeg:'.jpeg"}}',
          imagegif:'.gif"}}', 
        }
   },
  props: {
    product: {
      type: Object,
      default: () => ({})
    },
    customOptions: {
      type: Object,
      default: () => ({})
    },
    reviews: {
      type: Array,
      default: () => []
    }
  },
  computed: {
    reviewsCount () {
      return this.reviews.length
    }
  },
  mounted () {
    this.productDescription  = this.product.description.replace(this.imagemediapath, (config.externalCheckout.cmsUrl+"/media/wysiwyg"));   
    this.productDescription  = this.productDescription.replace(this.imagepng, ".png");   
    this.productDescription  = this.productDescription.replace(this.imagejpg, ".jpg");   
    this.productDescription  = this.productDescription.replace(this.imagejpeg, ".jpeg");   
    this.productDescription  = this.productDescription.replace(this.imagegif, ".gif");   
  },
  methods: {
    openReviewsTab () {
      this.$store.commit('ui/setVideoProductTab', false);
      this.$store.commit('ui/setDescriptionProductTab', false);
      this.$store.commit('ui/setReviewProductTab', true);
       
      const reviewsEl = document.querySelector('#m-product-additional-info');
      if (!reviewsEl) return;

      const scrollTop = window.pageYOffset || document.documentElement.scrollTop;
      createSmoothscroll(scrollTop, scrollTop + reviewsEl.getBoundingClientRect().top);
    },
    openDescriptionTab() {
      this.$store.commit('ui/setReviewProductTab', false);
      this.$store.commit('ui/setVideoProductTab', false);
      this.$store.commit('ui/setDescriptionProductTab', true);
      
      const reviewsEl = document.querySelector('#m-product-additional-info');
      if (!reviewsEl) return;

      const scrollTop = window.pageYOffset || document.documentElement.scrollTop;
      createSmoothscroll(scrollTop, scrollTop + reviewsEl.getBoundingClientRect().top);
    },
    showOrHideBlocksBasedOnBFTG (product) {
      const availableManufactures = config.availableManufactures;
      return availableManufactures.includes(product.manufacturer);
    }
  }
};
</script>
<style lang="scss" scoped>
@import "~@storefront-ui/shared/styles/helpers/breakpoints";
@import "~@storefront-ui/shared/styles/helpers/typography";

.product {
  &__header {
    display: flex;
    justify-content: space-between;
    @include for-desktop {
      margin: 0 auto;
    }
  }
  &__drag-icon {
    animation: moveicon 1s ease-in-out infinite;
  }
  &__price-and-rating {
    margin: var(--spacer-xs) var(--spacer-sm) var(--spacer-base);
    align-items: center;
    @include for-desktop {
      display: flex;
      justify-content: space-between;
      margin: var(--spacer-sm) 0 var(--spacer-lg) 0;
    }
  }
  &__description {
    color: var(--c-link);
    @include font(
      --product-description-font,
      var(--font-light),
      var(--font-base),
      1.6,
      var(--font-family-primary)
    );
  }
}

@keyframes moveicon {
  0% {
    transform: translate3d(0, 0, 0);
  }
  50% {
    transform: translate3d(0, 30%, 0);
  }
  100% {
    transform: translate3d(0, 0, 0);
  }
}
</style>
