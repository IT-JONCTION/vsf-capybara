<template>
  <div class="garantie-product d-flex">
            <SfImage
            src="/assets/images/big-garantie-2years-icon.png"
            alt="garantie-icon"
            class="sf-heade"
             />
        <div class="description">
            <p class="text-primary">
              <b>Product warranty</b>
            </p>
            <p>This product is under warranty for two years from the date of purchase.</p>
            <SfLink target="_blank" :link="link">
                Learn more
            </SfLink>
        </div>
  </div>
</template>

<script>
import { SfLink, SfImage } from "@storefront-ui/vue";
export default {
  components: {
    SfLink,
    SfImage,
  },
  data() {
    return {
      link: "https://www.capitools.com/fr/content/40-ma-garantie-swap",
    };
  },
};
</script>

<style lang="scss" scoped>
@import "~@storefront-ui/shared/styles/helpers/breakpoints";
    .garantie-product{
        padding: 15px;
        border: 1px solid var(--_c-gray-accent);
        border-radius: 5px;
        width: fit-content;
        margin: 15px 0;
        @include for-mobile{
          display: inline-flex;
          width: 43vw;
          margin: 15px 0;
          align-items: center;
          vertical-align: text-bottom;
        }
        @media screen and (max-width: 767px){
             width: auto;
            display: flex;
        }
        .sf-image{
            min-width: auto;
            width: 160px;
           
            @include for-desktop{
               min-width: 120px;
            }
        }
        p{
            margin: 0;
            font-size: 12px;
            &.text-primary{
                color: var(--_c-green-primary-lighten)!important;
                margin-bottom: 7px;
            }
        }
        .description{
            margin: 0 15px;
            a{
               font-family: 'Poppins-Light';
               font-style: italic;
               font-size: 12px; 
               color: var(--_c-green-primary-lighten)!important;
               text-decoration: none;
                font-weight: 600;
            }
        }
    }
</style>