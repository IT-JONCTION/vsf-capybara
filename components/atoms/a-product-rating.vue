<template>
  <div class="a-product-rating" @click="$emit('click')">
    <div class="product__rating">
      <SfRating :score="score" :max="max" />
      <a v-if="!!reviews" href="#" class="product__count">
        ({{ reviews.length }})
      <span> Customer reviews</span>
      </a>
    </div>
    <SfButton class="sf-button--text">
      <slot />
    </SfButton>
  </div>
</template>

<script>
import { SfRating, SfButton } from '@storefront-ui/vue';
export default {
  components: {
    SfRating,
    SfButton
  },
  props: {
    reviews: {
      type: Array,
      default: () => []
    }
  },
  data () {
    return {
      max: 5
    }
  },
  computed: {
    score () {
      var ratingStarCount = 0;
      if(this.count > 0){
         var tempRatingStarCount =  0;
         var tempTotalRatingCount = 0;
        for (let iLoop = 0; iLoop < this.count; iLoop++) {
          tempTotalRatingCount = tempTotalRatingCount + this.reviews[iLoop].rating.length;
          for (let jLoop = 0; jLoop < this.reviews[iLoop].rating.length; jLoop++) {
            tempRatingStarCount = tempRatingStarCount + this.reviews[iLoop].rating[jLoop].value;
           }   
        }
        if( tempRatingStarCount > 0 && tempTotalRatingCount > 0){
          ratingStarCount = tempRatingStarCount/tempTotalRatingCount;
        }
      }
      console.info(ratingStarCount);
      return ratingStarCount;
      //const allScore = this.reviews.reduce((sum, rev) => (sum + (rev.rating || 0)), 0)
      // return allScore ? allScore / this.count : allScore
    },
    count () {
      return this.reviews.length
    }
  }
};
</script>

<style lang="scss" scoped>
.product {
  &__rating {
    display: flex;
    align-items: center;
    margin: var(--spacer-xs) 0 0 0;
  }
  &__count {
    color: var(--_c-green-primary-lighten);
    text-decoration: none;
    margin: 0 10px;
    font-size: 13px;
    padding: 0;
    &:hover{
      text-decoration: underline;
    }
  }
}

.sf-rating{
  &__icon{
    color: var(--_c-orange-primary);
  }
}
</style>
