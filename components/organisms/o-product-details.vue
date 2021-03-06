<template>
  <div class="o-product-details product" itemprop="offers" itemscope itemtype="http://schema.org/Offer">
    <meta itemprop="priceCurrency" :content="$store.state.storeView.i18n.currencyCode">
    <meta itemprop="price" :content="parseFloat(product.price_incl_tax).toFixed(2)">
    <meta itemprop="availability" :content="availability">
    <meta itemprop="url" :content="product.url_path">
    <MProductGallery
      :offline-image="offlineImage"
      :gallery="gallery"
      :configuration="productConfiguration"
      :product="product"
    />
    <div class="product__info">
      <div class="d-flex align-start">
        <MProductShortInfo
          :product="product"
          :custom-options="productCustomOptions"
          :reviews="reviews"
        />
        <div class="quantity-section">

          <MProductOptionsConfigurable
            v-if="product.type_id =='configurable'"
            :product="product"
            :configuration="productConfiguration"
          />
          <MProductOptionsGroup
            v-if="product.type_id =='grouped'"
            :product-options="product.product_links"
          />
         
          <MProductAddToCart
            class="product__add-to-cart"
            :product="product"
            :stock="productStock"
          />
          <div class="product_manual" v-if="isShowProductManualTab"> 
            <template>
              <img :src="'/assets/images/ownermanual.png'"
                    class="image_owner_manual" />
              <SfLink :link="productManualPath" target="_blank">
                View the product manual
              </SfLink>
            </template>
          </div>
        </div>
 
      </div>
    </div>
  </div>
</template>
<script>
import get from 'lodash-es/get'
import config from 'config';
import { mapGetters, mapActions } from 'vuex';
import MProductGallery from 'theme/components/molecules/m-product-gallery';
import MProductShortInfo from 'theme/components/molecules/m-product-short-info';
import MProductAddToCart from 'theme/components/molecules/m-product-add-to-cart';
import MProductOptionsConfigurable from 'theme/components/molecules/m-product-options-configurable';
import MProductOptionsBundle from 'theme/components/molecules/m-product-options-bundle';
import MProductOptionsCustom from 'theme/components/molecules/m-product-options-custom';
import MProductOptionsGroup from 'theme/components/molecules/m-product-options-group';
import { ModalList } from 'theme/store/ui/modals';
import { SfLink } from '@storefront-ui/vue';

export default {
  components: {
    MProductGallery,
    MProductShortInfo,
    MProductAddToCart,
    MProductOptionsConfigurable,
    MProductOptionsBundle,
    MProductOptionsCustom,
    MProductOptionsGroup,
    SfLink
  },
  data() {
    return { 
        productManualPath: '', 
      }
  },
  props: {
    product: {
      type: Object,
      default: () => ({})
    },
    productGallery: {
      type: Array,
      default: () => []
    },
    productConfiguration: {
      type: Object,
      default: () => ({})
    },
    productCustomOptions: {
      type: Object,
      default: () => ({})
    },
    productAttributes: {
      type: Array,
      default: () => []
    },
    productStock: {
      type: Object,
      default: () => ({})
    }
  },
  computed: {
    offlineImage () {
      const width = config.products.thumbnails.width;
      const height = config.products.thumbnails.height;
      return {
        mobile: {
          url: this.getThumbnail(this.product.image, width, height),
          alt: this.product.name
        },
        desktop: {
          url: this.getThumbnail(this.product.image, width, height),
          alt: this.product.name
        }
      };
    },
    gallery () {
      return this.productGallery.map(imageObject => ({
        id: imageObject.id,
        mobile: {
          url: imageObject.src,
          alt: this.product.name
        },
        desktop: {
          url: imageObject.src,
          alt: this.product.name
        }
      }));
    },
    reviews () {
      const baseReviews = get(this.$store.state.review, 'items.items', [])
      return baseReviews.map((review) => ({
        author: review.nickname,
        date: review.created_at,
        message: `${review.title}: ${review.detail}`,
        rating: review.ratings
       }))
    },
    availability () {
      return this.product.stock && this.product.stock.is_in_stock ? 'InStock' : 'OutOfStock'
    },
    sizeOption () {
      return get(this.productConfiguration, 'size', false)
    },
    isShowProductManualTab(){
      return this.product.product_manual && this.product.product_manual.length > 0  ? true : false ;
    }
  },
  mounted () {
     this.productManualPath   = this.product.product_manual && this.product.product_manual.length > 0 ? this.product.product_manual[0].pdf_path : '';
  },
  methods: {
    ...mapActions('ui', {
      openModal: 'openModal'
    }),
    openSizeGuide () {
      this.openModal({ name: ModalList.SizeGuide })
    }
  }
};
</script>
<style lang="scss" scoped>
@import "~@storefront-ui/shared/styles/helpers/breakpoints";

.product {
  flex-direction: column;
  @include for-desktop {
    display: flex;
    flex-direction: row;
  }
  &__info {
    margin: var(--spacer-sm) auto var(--spacer-xs);
    @include for-desktop {
      margin: 0 ;
      max-width: calc(100% - 550px);
    }
  }
  &__add-to-cart {
    margin:  0;
    @include for-desktop {
      
    }
  }
  &__guide,
  &__compare,
  &__save {
    display: block;
    margin: var(--spacer-xl) 0 var(--spacer-base) auto;
  }
}

.quantity-section{
  .product__add-to-cart{
     
      margin: 0 8px;
      margin-bottom: 15px;
      padding: 15px;
      border: 1px solid var(--_c-green-primary-lighten);
      border-radius: 3px;
      @include for-mobile{
        width: -webkit-fill-available;
        min-width: 260px;
        width: 100%;
      }
    }
    
  }

.section {
  border-bottom: 1px solid #f1f2f3;
  padding-bottom: 10px;
  @include for-desktop {
    border: 0;
    padding-bottom: 0;
  }
}
</style>
