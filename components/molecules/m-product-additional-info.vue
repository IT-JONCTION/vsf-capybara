<template>
  <SfTabs
    class="m-product-additional-info product__tabs"
    id="m-product-additional-info"
    ref="productTabs"  
    @toggle="onToggle"
  >
    <SfTab :title="$t('Description')" ref="descriptionTab"> 
      <div class="description" >
        <p itemprop="description" v-html="this.productDescription"></p>
      </div>
    </SfTab>
    <SfTab :title="$t('Features')" v-if="product.feature_bullets"> 
      <div class="features" >
        <p itemprop="features" v-html="product.feature_bullets"> </p>
      </div>
    </SfTab>
    <SfTab :title="$t('Specs')" v-if="attributes.length"> 
      <div class="specs" >
        <AProductAttribute
          v-for="(attribute, i) in attributes"
          :key="i"
          :product="product"
          :attribute="attribute"
          class="product__property" 
        />
      </div>
    </SfTab>
    <SfTab :title="$t('Premium')" v-if="product.include_pdi"> 
      <div class="premium" >
        <cms-block :identifier="'pdi-text-capitools'" />
      </div>
    </SfTab>
    <SfTab :title="$t('Reviews')" ref="reviewTab"> 
      <div class="review-header">
        <SfHeading
          :title="$t(this.productReviewCount)"
          :level="3"
          class="sf-heading--left"
        />
        <AProductRating
          @click="handleOpenReviewModal"
          :reviews="reviews"
        >
            {{ $t('Write a review') }}
        </AProductRating>
      </div>
     
      <SfDivider v-show="reviewsCount" />
      <MReviewList v-show="reviewsCount" :reviews="reviews" :visible=10 />
    </SfTab>
    <SfTab :title="$t('Product Manual')" v-if="isShowProductManualTab"> 
      <div class="product_manual" >
        <template>
           <img :src="'/assets/images/ownermanual.png'"
                class="image_owner_manual" />
          <SfLink :link="productManualPath" target="_blank">
            View the product manual
          </SfLink>
        </template>
      </div>
    </SfTab>
    <SfTab :title="$t('Video')" ref="videoTab"  v-if="isShowVideoTab">  
      <div class="video-grid">
        <div class="video-item" v-if="product.youtube_video_code_one">
          <youtube v-if="product.youtube_video_code_one"
                   :video-id="productVideoIdOne"
                   :player-vars="playerVars"
                   @playing="playing"
          />
        </div>
        <div class="video-item" v-if="product.youtube_video_code_two">
          <youtube :video-id="productVideoIdTwo"
          />
        </div>
        <div class="video-item" v-if="product.youtube_video_code_three">
          <youtube :video-id="productVideoIdThree"
          />
        </div>
      </div>
    </SfTab> 
  </SfTabs>
</template>

<script>
import { ModalList } from 'theme/store/ui/modals';
import { mapState, mapActions } from 'vuex';
import { SfHeading, SfTabs, SfDivider, SfLink } from '@storefront-ui/vue';
import AProductRating from 'theme/components/atoms/a-product-rating';
import AProductAttribute from 'theme/components/atoms/a-product-attribute';
import MReviewList from 'theme/components/molecules/m-review-list'; 
import CmsBlock from 'theme/components/core/blocks/Cms/Block';
import config from 'config';
import Vue from 'vue'
import VueYoutube from 'vue-youtube'
Vue.use(VueYoutube)

export default {
  name: 'MProductAdditionalInfo',
  components: {
    SfHeading,
    SfTabs,
    AProductRating,
    SfDivider, 
    SfLink,
    AProductAttribute,
    MReviewList,
    CmsBlock,
  },
   mixins: [CmsBlock],
  data() {
   return {
     productReviewCount: '',
      productVideoIdOne: '',
      productVideoIdTwo: '',
      productVideoIdThree: '', 
      productManualPath: '',
      productDescription:'',
      imagemediapath:'{{media url="wysiwyg',
      imagepng:'.png"}}',
      imagejpg:'.jpg"}}',
      imagejpeg:'.jpeg"}}',
      imagegif:'.gif"}}',
      playerVars: {
        autoplay: 1
      }
    }
   },
   
  props: {
    reviews: {
      type: Array,
      default: () => []
    },
    attributes: {
      type: Array,
      default: () => []
    },
    product: {
      type: Object,
      default: () => ({})
    }
  },
  computed: {
    ...mapState({
      isReviewProductTab: state => state.ui.isReviewProductTab,
      isVideoProductTab: state => state.ui.isVideoProductTab,
      isDescriptionProductTab: state => state.ui.isDescriptionProductTab
    }),
    reviewsCount () {
      this.productReviewCount =   this.reviews.length + " Reviews";
      return this.reviews.length;
    }, 
    isShowVideoTab(){
      return this.product.youtube_video_code_one != null || this.product.youtube_video_code_two != null || this.product.youtube_video_code_three != null  ? true : false ;
    },
    isShowProductManualTab(){
      return this.product.product_manual && this.product.product_manual.length > 0  ? true : false ;
    }
  },
  mounted () {
    this.productVideoIdOne   = this.product.youtube_video_code_one != null ? this.$youtube.getIdFromUrl(this.product.youtube_video_code_one) : '';
    this.productVideoIdTwo   = this.product.youtube_video_code_two != null? this.$youtube.getIdFromUrl(this.product.youtube_video_code_two) : '';
    this.productVideoIdThree = this.product.youtube_video_code_three != null? this.$youtube.getIdFromUrl(this.product.youtube_video_code_three) : '';
    this.productManualPath   = this.product.product_manual && this.product.product_manual.length > 0 ? this.product.product_manual[0].pdf_path : '';
    this.productDescription  = this.product.description.replace(this.imagemediapath, (config.externalCheckout.cmsUrl+"/media/wysiwyg"));   
    this.productDescription  = this.productDescription.replace(this.imagepng, ".png");   
    this.productDescription  = this.productDescription.replace(this.imagejpg, ".jpg");   
    this.productDescription  = this.productDescription.replace(this.imagejpeg, ".jpeg");   
    this.productDescription  = this.productDescription.replace(this.imagegif, ".gif");   
  },
  watch: {
    isReviewProductTab (value) {
      if (value && !this.$refs.reviewTab.isActive) {
        this.$refs.productTabs.toggle(this.$refs.reviewTab._uid);
      }
    },
    isVideoProductTab (value) {
      if (value && !this.$refs.videoTab.isActive) {
        this.$refs.productTabs.toggle(this.$refs.videoTab._uid);
      }
    },
    isDescriptionProductTab (value) {
      if (value && !this.$refs.descriptionTab.isActive) {
        this.$refs.productTabs.toggle(this.$refs.descriptionTab._uid);
      }
    }
  },
  beforeDestroy () {
    this.$store.commit('ui/setReviewProductTab', false);
    this.$store.commit('ui/setVideoProductTab', false);
    this.$store.commit('ui/setDescriptionProductTab', false);
  }, 
  methods: {
    ...mapActions('ui', {
      openModal: 'openModal'
    }),
    handleOpenReviewModal () {
      this.openModal({ name: ModalList.ReviewPwa, payload: this.product.id })
    },
    onToggle (id) { 
      this.$store.commit('ui/setReviewProductTab', id === this.$refs.reviewTab._uid);
      this.$store.commit('ui/setVideoProductTab', id === this.$refs.videoTab._uid);
      this.$store.commit('ui/setDescriptionProductTab', id === this.$refs.descriptionTab._uid);
    }, 
  }
};
</script>

<style lang="scss" scoped>
@import "~@storefront-ui/shared/styles/helpers/breakpoints";

.product {
  &__tabs {
    margin: var(--spacer-lg) auto var(--spacer-2xl);
    @include for-desktop {
      margin-top: var(--spacer-2xl);
      --tabs-content-tab-padding: 3.5rem 0 0 0;
    }
  }
  &__property {
    margin: var(--spacer-base) 0;
  }
}

[itemprop="description"] > *:first-child {
  margin-top: 0;
}

.review-header {
  display: flex;
  justify-content: space-between;
}
</style>
