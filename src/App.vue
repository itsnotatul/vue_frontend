<template>
  <div id="app">
    <h1>Payload Overwrite UI</h1>
    <form @submit.prevent="sendPayload">
      <div>
        <label for="event">Select Event:</label>
        <select v-model="selectedEvent" id="event" @change="loadEventFields">
          <option v-for="event in events" :key="event.name" :value="event.name">
            {{ event.name }}
          </option>
        </select>
      </div>

      <!-- Show the fields for the selected event -->
      <div v-if="fields.length">
        <h3>Configure Payload</h3>
        <div v-for="(field, index) in fields" :key="index" class="field-row">
          <label :for="field.name">{{ field.name }}:</label>
          <input
            type="text"
            v-model="field.value"
            :id="field.name"
            :placeholder="field.defaultValue"
          />
        </div>
      </div>

      <!-- Select JSON Structure -->
      <div>
        <label>Select JSON Structure:</label>
        <label><input type="radio" value="standard" v-model="jsonStructure" /> Standard</label>
        <label><input type="radio" value="escaped" v-model="jsonStructure" /> Escaped</label>
        <label><input type="radio" value="single-quoted" v-model="jsonStructure" /> Single-Quoted</label>
      </div>

      <!-- Select Scenario Type -->
      <div>
        <label for="scenario">Scenario Type:</label>
        <select v-model="scenarioType" id="scenario">
          <option value="positive">Positive</option>
          <option value="negative">Negative</option>
        </select>
      </div>

      <!-- Submit Button -->
      <button type="submit">Send</button>
    </form>
  </div>
</template>

<script>
export default {
  data() {
    return {
      events: [
        {
          name: "Event1",
          fields: [
            { name: "Field1", defaultValue: "Default1" },
            { name: "Field2", defaultValue: "Default2" },
          ],
        },
        {
          name: "Event2",
          fields: [
            { name: "FieldA", defaultValue: "DefaultA" },
            { name: "FieldB", defaultValue: "DefaultB" },
          ],
        },
      ],
      selectedEvent: "",
      fields: [], // This will hold the fields for the selected event
      jsonStructure: "standard", // Default JSON structure
      scenarioType: "positive", // Default scenario type
    };
  },
  methods: {
    loadEventFields() {
      const event = this.events.find((e) => e.name === this.selectedEvent);
      if (event) {
        // Load the fields and set their default values
        this.fields = event.fields.map((field) => ({
          name: field.name,
          value: field.defaultValue, // Set the default value initially
          defaultValue: field.defaultValue,
        }));
      } else {
        this.fields = []; // Clear fields if no event is selected
      }
    },
    async sendPayload() {
      const payload = this.fields.reduce((acc, field) => {
        acc[field.name] = field.value;
        return acc;
      }, {});

      const requestBody = {
        event: this.selectedEvent,
        payload: payload,
        scenario: this.scenarioType,
      };

      try {
        const response = await fetch("http://localhost:8080/overwrite", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(requestBody),
        });
        await response.json();
        alert("Payload overwritten successfully!");
      } catch (error) {
        console.error(error);
        alert("Failed to send payload.");
      }
    },
  },
};
</script>

<style>
.field-row {
  margin-bottom: 10px;
}
</style>
