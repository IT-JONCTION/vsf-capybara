<template>
  <div class="m-product-add-to-cart">
    <MProductOptionsBundle
      v-if="product.bundle_options && product.bundle_options.length > 0"
      :product="product"
    />
    <MProductOptionsCustom
      v-else-if="product.custom_options && product.custom_options.length > 0"
      :product="product"
    />
    <div v-show="this.stock.isLoading">
      <div class="show-loader-stock-api"></div>
    </div>
    <SfAlert
      v-show="!this.stock.isLoading"
      class="alert"
      :message="alert.message"
      :type="alert.type"
    /> 
    <SfAddToCart v-model="qty">
      <template #add-to-cart-btn>
        <AAddToCart
          class="sf-add-to-cart__button"
          :qty="qty"
          :product="product"
          :disabled="isAddToCartDisabled"
        />
      </template>
      <template #quantity-select-input>
          <AProductPrice
              v-if="product.type_id !== 'grouped'"
              :product="product"
              :custom-options="customOptions"
          />
          <AQuantityInfo
            :product="product"/> 
          <div class="quantity d-flex align-center">
              <label>Quantity :</label>
              <AProductQuantity
                class="sf-add-to-cart__select-quantity"
                v-if="product.type_id !== 'grouped' && product.type_id !== 'bundle'"
                v-model="qty"
                :max-quantity="stock.max"
                :loading="stock.isLoading"
                :unlimit-quantity="!isSimpleOrConfigurable"
                @input="$emit('input', $event)"
                @error="handleQuantityValidationError"
              />
              <span v-else>&nbsp;</span>
            </div>
      </template>
    </SfAddToCart>
     <div class="payment-security d-flex align-center justify-center"> 
        <SfImage
            src="/assets/icons/paiement-securise-icon.png"
            :alt="$t(defaultTitle)"
            class="icon"
            />
        <p>100% secure payment</p>
    </div>
  </div>
</template>
<script>
import { onlineHelper } from '@vue-storefront/core/helpers';
import { SfAddToCart, SfAlert, SfPrice, SfImage } from '@storefront-ui/vue';
import AProductQuantity from 'theme/components/atoms/a-product-quantity';
import AQuantityInfo from 'theme/components/atoms/a-quantity-info';
import AAddToCart from 'theme/components/atoms/a-add-to-cart';
import AProductPrice from 'theme/components/atoms/a-product-price';
import MProductOptionsCustom from 'theme/components/molecules/m-product-options-custom';
import MProductOptionsBundle from 'theme/components/molecules/m-product-options-bundle';


export default {
  name: 'MProductAddToCart',
  components: {
    SfAddToCart,
    SfAlert,
    AProductQuantity,
    AQuantityInfo,
    AAddToCart,
    SfPrice,
    SfImage,
    AProductPrice,
    MProductOptionsCustom,
    MProductOptionsBundle
  },
  data () {
    return {
      qty: 1,
      qtyValidationError: '',
      customOptions: {
      type: Object,
      default: () => ({})
    },
      defaultTitle: ''
    };
  },
  props: {
    product: {
      type: Object,
      required: true,
      default: () => ({})
    },
    stock: {
      type: Object,
      default: () => ({})
    }
  },
  computed: {
    isOnline () {
      return onlineHelper.isOnline;
    },
    isAddToCartDisabled () {
      return !!this.qtyValidationError || this.stock.isLoading || !this.isAvailable
    },
    isAvailable () {
      return !this.isOnline || !!this.stock.max || !this.stock.manageQuantity || !this.isSimpleOrConfigurable || this.stock.backorder
    },
    isSimpleOrConfigurable () {
      return ['simple', 'configurable'].includes(
        this.product.type_id
      );
    },
    alert () {
      if (this.qtyValidationError) {
        return {
          type: 'danger',
          message: this.qtyValidationError
        }
      }
      if (!this.isAvailable) {
        return {
          type: 'danger',
          message: this.$t('Selected variant is out of stock')
        }
      }
      return false
    }
  },
  methods: {
    handleQuantityValidationError (error) {
      this.qtyValidationError = error
    }
  }
};
</script>

<style lang="scss" scoped>
.alert {
  margin-bottom: var(--spacer-base)
}

.sf-add-to-cart{
  flex-direction: column; 
}
</style>
