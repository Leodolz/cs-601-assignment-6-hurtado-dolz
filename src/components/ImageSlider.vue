<template>
  <!-- We use transition-group as a base from our image carousel. With CSS we can make a good implementation -->
  <!-- We add a callback before a current item leaves and we add a name depending on the direction it will go -->
  <transition-group tag="div" class="slide-group"
                    :name="transitionName"
                    @before-leave="sgBeforeLeave">
    <!-- We use an inner container for having a nice presentation with padding and margins -->
    <div :key="current" class="slide">
      <!-- This is a row icon drew using polyline and wrapped by svg. It acts as a left navigation button
      this is why the function called when clicked, gives a negative direction -->
      <svg viewBox="0 0 13 20" id="previous-arrow" class="nav" @click="move(-1)">
        <polyline points="10,3 3,10 10,17"></polyline>
      </svg>

      <!-- Right arrow drew in the same way as above with a positive direction when clicked -->
      <svg viewBox="0 0 13 20" id="next-arrow" class="nav nav--next" @click="move(1)">
        <polyline points="10,3 3,10 10,17" transform="rotate(180 6.5,10)"></polyline>
      </svg>
      <!-- Two images but using v-if which will select one or the other depending on the current slide's isEarlier
       attribute. -->
      <!-- As this first image represents the earlierImage it has a mouseOut event and renders the earlierImage from
      the slide -->
      <img v-if="slides[current].isEarlier" :src="slides[current].earlierImage" @mouseout="mouseOut()"
           :alt="slides[current].earlierImageAlt" @click="imgClick(1)" class="hovered"/>
      <!-- As this second image with v-else represents the currentImage, it has a mouseOver event and renders the
       current image from the slide. This is to fulfill the requirement of hovering and showing an early image and
       show the present image when not hovering such as the assignment asks -->
      <img v-else :src="slides[current].image" :alt="slides[current].imageAlt"
           @click="imgClick(1)" @mouseenter="mouseOver()"/>
      <!-- Description section which will contain the description of the early image, it is also rendered conditionally
      and share the same title as it is a before/after comparison -->
      <section v-if="slides[current].isEarlier" id="early-description">
        <h3>{{ slides[current].title }}: Before</h3>
        <p v-for="paragraph in slides[current].earlierContent.split('\n')" :key="paragraph">{{ paragraph }}</p>
      </section>
      <!-- Description section that contains the current image using v-else as it is conditional of the isEarlier
       attribute from the current slide -->
      <section v-else id="current-description">
        <h3>{{ slides[current].title }}: Currently</h3>
        <p v-for="paragraph in slides[current].content.split('\n')" :key="paragraph">{{ paragraph }}</p>
      </section>
    </div>
  </transition-group>
  <!-- Import of settings surrounding it with a container for better visibility -->
  <section id="settings">
    <!-- Notice how we implement the emitted callback announced inside this component which is onSelected -->
    <ImageSettings @onSelected="selectionClick"/>
  </section>
  <br/>
  <!-- This section is done only for preloading all images and have smoother transitions-->
  <section id="preloader">
    <img v-for="slide in slides" :src="slide.image" :key="slide.id"/>
  </section>
</template>
<script type="module">
import ImageSettings from "@/components/ImageSettings.vue";

export default {
  // Define the name of the component
  name: 'ImageSlider',
  // Declared the components used here
  components: {
    ImageSettings,
  },
  // Declare the methods for usage in this component
  methods: {
    // Method when the mouse is hovering the image and show the earlier image
    mouseOver: function () {
      // If the option of hovering is on, it shows the earlier image when hovered
      if (this.byHoover) {
        this.slides[this.current].isEarlier = true;
      }
    },
    // Method when the mouse is done hovering the image, it should stop displaying the early image
    mouseOut: function () {
      // If the hover option is on, stop displaying early image
      if (this.byHoover) {
        this.slides[this.current].isEarlier = false;
      }
    },
    // Method for interpreting the option selected for hovering or click feature
    selectionClick: function (selection) {
      this.byHoover = this.selectionMap[selection];
    },
    // Method for toggling the slide's early image showing
    toggleSlide: function () {
      this.slides[this.current].isEarlier = !this.slides[this.current].isEarlier;
    },
    // Method when the image is clicked, it has different behavior depending on the hovering feature
    imgClick: function (direction) {
      if (!this.byHoover) {
        this.toggleSlide();
      } else {
        this.move(direction);
      }
    },
    move(direction) {
      this.direction = direction;
      // Move forward or backwards according to direction, if we go before the first slide, we translate it to the last
      // row for having a looped gallery
      if (this.current + direction < 0) {
        this.current = this.slides.length - 1;
      } else {
        // If we are heading upwards, we simply use modulo operator if we appear to go further than the limit
        this.current = (this.current + direction) % this.slides.length;
      }
    },

    // As an element from this group will leave, the width and height are modified to a consistent smaller, so it doesn't
    // hover by accident and have a smooth transitioning
    sgBeforeLeave(element) {
      let rect = element.getBoundingClientRect();
      element.style.width = (rect.right - rect.left) + 'px';
      element.style.height = (rect.bottom - rect.top) + 'px';
    }
  },

  // Define the name of the transition if it will go right or left, for css styling
  computed: {
    transitionName() {
      return 'sg-' +
          (this.direction > 0 ? 'right' : 'left');
    }
  },
  // Define data available to access and/or modify in our component
  data() {
    return {
      // Direction is 0 when starting the program, it will change if it goes right or left
      direction: 0,
      // This indicates the index of the currently presented image, we point to the first image by default
      current: 0,
      // Map for interpreting the selection from the settings component, it will indicate if hover feature is on or off
      selectionMap: {
        hover: true,
        click: false,
      },
      // Boolean variable that defines if hovering feature is on or off
      byHoover: true,
      // Slides array for display in carousel
      slides: [
          // Each item has the same attributes as below
        {
          // The title of the slide
          title: 'Fidel Anze Park',
          // The current image source
          image: require('@/assets/fidel-anze-after.jpeg'),
          // The current image alt
          imageAlt: 'Image of the Fidel Anze park currently, it shows a park with different trees and inner walkways',
          // The current image description
          content: "This park is commonly browsed by people all day along and is popular for photo-shooting for " +
              "different kinds of events.\nThis is also very common to be visited by people with their pets, usually " +
              "dogs.",
          // The earlier image source
          earlierImage: require('@/assets/fidel-anze-before.jpeg'),
          // The earlier image description
          earlierContent: "This park was not maintained in the past. It had no usage from the townsfolk and had " +
              "very seldom trees.\nThis park had no walkways as its grass was very tall and not being cut as there " +
              "was no investment from the city.",
          // The earlier image alt
          earlierImageAlt: 'Image of the Fidel Anze park in the past, it shows a park with tall grass and no walkways',
          // The boolean that indicates if the slide should display the earlier image or not
          isEarlier: false,
        },
        {
          title: 'Plaza 4 de Noviembre',
          image: require('@/assets/plaza-4-de-noviembre-after.jpeg'),
          imageAlt: '',
          content: "This plaza has been renewed and now had better fountains and contains better security to ensure " +
              "a safe walk for people nearby at all hours.\nThis place is usually more crowded on the evenings.",
          earlierImage: require('@/assets/plaza-4-de-noviembre-before.jpeg'),
          earlierContent: "This plaza was an abandoned park which was usually avoided by people nearby as it " +
              "had almost no security and it had no investment from the major's office.\nIts characteristic fountain " +
              "did not work and it contained a kiosk for buying snacks. This place was mostly surrounded by houses " +
              "and some buildings.",
          earlierImageAlt: 'Alt 2',
          isEarlier: false,
        },
        {
          title: 'Plaza 14 de Septiembre',
          image: require('@/assets/plaza-14-septiembre-after.jpeg'),
          imageAlt: '',
          content: "This plaza is used by people living nearby for taking a stroll and the government usually " +
              "organizes informing events at daylight.\nThis place is also nearby the major's office",
          earlierImage: require('@/assets/plaza-14-septiembre-before.jpeg'),
          earlierContent: "This plaza is where the first protests against colonies were done and where the dream " +
              "of a republic started.\nThis plaza celebrates the oldest foundation against colonies in Bolivia",
          earlierImageAlt: '',
          isEarlier: false,
        },
        {
          title: 'Felix Capriles Stadium',
          image: 'https://www.skyscrapercity.com/attachments/25e3b1d519a0f7087a349c154472482add2657d4-jpg.291150',
          imageAlt: 'An image of the felix capriles stadium nowadays, it contains various colors and the letters ' +
              'of Cochabamba on white are visible from a great distance',
          content: "This stadium is very crowded when the local team 'Wilsterman' plays, as people from Cochabamba " +
              "are usually rooting for this team.\nThis place is built for a diverse set of olympic disciplines " +
              "and is regularly used by professional athletes for competitions and preparing for tournaments at " +
              "a national scale.",
          earlierImage: 'https://i.goalzz.com/?i=katkotati%2Fstade%2Fsud_amerique%2Festadio+felix.jpg',
          earlierContent: "This is the stadium before its restoring work, this picture shows how it was already " +
              "in need for maintenance and shows a less diverse set of colors.",
          earlierImageAlt: '',
          isEarlier: false,
        },
        {
          title: 'Alalay Lake',
          image: 'https://www.pub.eldiario.net/noticias/2016/2016_04/nt160429/f_2016-04-29_17.jpg',
          imageAlt: 'An image of how the Alalay lake located in Cochabamba looks now, it shows a picture of a ' +
              'lae surrounded by city and houses',
          content: "This lake is a place with a nice view and is surrounded by both a walkway and a bike way " +
              "as it has a big radius, many people use it for doing exercise while having a good view.\nOn its " +
              "surroundings there is a construction covering a big area that is used for hosting the annual fair " +
              "which lasts 10 days and is mostly crowded on the nights. This construction is also used for hosting big " +
              "events such as concerts and big parties.",
          earlierImage: 'https://www.la-razon.com/wp-content/uploads/2022/11/15/10/alalay.png',
          earlierContent: "This lake used to cover a bigger area and had clearer waters. With urbanization that " +
              "made progress each year, came a harder way to maintain this lake which can get big areas " +
              "dried out during summer.",
          earlierImageAlt: 'This image shows Lake Alalay in the earlier days, it shows a clear lake area without ' +
              'urban constructions and depicting two fishermen going on a boat',
          isEarlier: false,
        },
        {
          title: 'Colón Plaza',
          image: 'https://i.pinimg.com/originals/3b/02/6b/3b026bee93ecc91b682a0a173329a9ea.jpg',
          imageAlt: "This image depicts a life-some plaza on daylight showing a big building with the colors of " +
              "yellow and white. It also depicts palm trees on the surroundings.",
          content: "This place is used as boulevard and a place for recreational events and activities. Being " +
              "located in the center of the city, it is a place where townsfolk can go pay for their taxes and " +
              "do other administrative activities regarding taxes.\nThis place hosts one big administrative " +
              "building for the major's offices and activities. This is also a place where people gather " +
              "when trying to get to an agreement with the Major.",
          earlierImage: 'https://i.ytimg.com/vi/Cwvp30EYMWM/hqdefault.jpg',
          earlierContent: "This place was even more crowded in the past as it was one of a handful of plazas " +
              "for doing different activities. Protests and gatherings were usually done here as this place " +
              "was still doing activities for working on the foundations of this city.",
          earlierImageAlt: '',
          isEarlier: false,
        }
      ]
    }
  },
};
</script>

<style scoped>
@import '@/styles/imageSliderStyle.css';
</style>