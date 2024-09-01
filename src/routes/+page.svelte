<script lang="ts">
  import GoogleMapsAutocomplete from '../components/GoogleMapsAutocomplete.svelte';
  import type { AutoCompleteType } from '../components/GoogleMapsAutocomplete.svelte';

  let values: AutoCompleteType[] = $state([]);

  const handleChange = (name: string, fieldsValues: AutoCompleteType[]) => {
    console.log('Name:', name, 'Values:', values);
    values = fieldsValues;
  }
</script>
<div class="header">
  <h1>Google Maps Autocomplete</h1>
  <h3>Form component using Google Maps Api</h3>
</div>
<div class="page">
  <div class="data">
    <h3>Values</h3>
    <div>
      <pre>{JSON.stringify(values, null, 2)}</pre>
    </div>
  </div>
  <div class="form">
    <div class="container">
      <GoogleMapsAutocomplete label={'Type your address'} onchange={(name, values) => {handleChange(name, values)}}/>
    </div>
  </div>
</div>
<style>
  .page {
    display: flex;
    flex-direction: row;
    align-items: center;
    width: 100%;
  }
  .header {
    flex: 1;
    text-align: center;
    margin-top: 20px;
  }
  .data {
    flex: .4;
    width: 40%;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    margin-top: 20px;
    overflow-x: auto;
  }
  .form {
    flex: .5;
    width: 50%;
    display: flex;
    justify-content: center;
    margin-top: 20px;
  }
</style>