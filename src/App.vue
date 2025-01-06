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
          <template v-if="!field.fields">
            <label :for="field.name">{{ field.name }}:</label>
            <input
              type="text"
              v-model="field.value"
              :id="field.name"
              :placeholder="field.defaultValue"
            />
          </template>

          <!-- Handle Nested Fields -->
          <template v-else>
            <div>
              <h4>{{ field.name }}</h4>
              <div class="nested-field" v-for="(nestedField, nestedIndex) in field.fields" :key="nestedIndex">
                <label :for="nestedField.name">{{ nestedField.name }}:</label>
                <input
                  type="text"
                  v-model="nestedField.value"
                  :id="nestedField.name"
                  :placeholder="nestedField.defaultValue"
                />
              </div>
            </div>
          </template>
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
        <select v-model="scenarioType" id="scenarioType">
          <option value="jay">jay</option>
          <option value="ray">ray</option>
        </select>
      </div>

      <!-- Select Testing Type -->
      <div>
        <label for="testing">Testing Type:</label>
        <select v-model="testingType" id="testingType">
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
            {
              name: "NestedField",
              fields: [
                { name: "NestedKey1", defaultValue: "NestedDefault1" },
                { name: "NestedKey2", defaultValue: "NestedDefault2" },
              ],
            },
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
      scenarioType: "jay", // Default scenario type
      testingType: "positive", // Default testing type
    };
  },
  methods: {
    loadEventFields() {
      const event = this.events.find((e) => e.name === this.selectedEvent);
      if (event) {
        this.fields = event.fields.map((field) => {
          if (field.fields) {
            // Initialize nested fields
            return {
              ...field,
              fields: field.fields.map((nestedField) => ({
                ...nestedField,
                value: nestedField.defaultValue,
              })),
            };
          }
          return { ...field, value: field.defaultValue };
        });
      } else {
        this.fields = []; // Clear fields if no event is selected
      }
    },
    async sendPayload() {
      const buildPayload = (fields) => {
        return fields.reduce((acc, field) => {
          if (field.fields) {
            // Recursively handle nested fields
            acc[field.name] = buildPayload(field.fields);
          } else {
            acc[field.name] = field.value;
          }
          return acc;
        }, {});
      };

      const payload = buildPayload(this.fields);

      const requestBody = {
        event: this.selectedEvent,
        payload: payload,
        scenarioType: this.scenarioType,
        testingType: this.testingType,
      };

      try {
        const response = await fetch("http://localhost:8081/overwrite", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(requestBody),
        });
        if (!response.ok) {
          const errorData = await response.json();
          alert(errorData.error || "Failed to send payloads.");
        } else {
          alert("Payloads overwritten successfully!");
        }
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
