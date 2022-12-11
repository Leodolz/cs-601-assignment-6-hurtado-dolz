<template>
  <h3>Image Settings</h3> <!-- Header for this component -->
  <p>How do you want to see the early image?</p> <!-- Question for the user for selecting one -->
  <!-- Selections for the settings defined by options. We use v-for to save some code and making this
  scalable as we can add more options in the future that can be rendered and trigger events without
  more added code -->
  <label :for="option.value" v-for="option in options" :key="option.value">
    <!-- We wrap input with label for making the v-for possible and we use radio buttons where v-model will
     be affected by the selection, which in this case is referencing to the selectedOption variable which
     was declared in data() -->
    <input type="radio" :id="option.value" :value="option.value" v-model="selectedOption"
           v-on:change="clickedInput" name="image-config">
    {{option.label}}
    <br/>
  </label>
</template>

<script>
export default {
  // We define the name of this component
  name: "ImageSettings",
  // We declare that this component emits this kind of event for informing parents that will make use of this
  emits: ["onSelected"],
  // In data, we return an object that contains the variables that can be accessed by "this" and can be modified
  // or simply read
  data() {
    return {
      selectedOption: "hover",
      options: [
        {
          value: "click",
          label: "By clicking the image"
        },
        {
          value: "hover",
          label: "By hovering the image"
        },
      ]
    }
  },
  // Here we declare the methods we may access and such have the access to the data
  methods: {
    clickedInput() {
      // With emit, we trigger what was defined by a parent when declaring this component, in this case is
      // the onSelected event that the parent should define, and we pass one parameter which is the selectedOption
      this.$emit('onSelected', this.selectedOption);
    },
  }
}
</script>
