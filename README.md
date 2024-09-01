# GoogleMapsAutocomplete

<img src="https://github.com/pigraphic/googlemapsautocomplete/blob/main/static/screenshot.png?raw=true" width="100%" alt="Example screenshot"/>


GoogleMapsAutocomplete is a Svelte component that integrates with the Google Maps Places API to provide address autocomplete functionality. This component is designed to be easily integrated into any Svelte application.

## Features

* **Address Autocomplete**: Provides suggestions for addresses as you type.
* **Customizable Fields**: Allows customization of the fields displayed, including address, city, postal code, province, country, and telephone.
* **Multiple Address**: stores multiple address in an array.
* **Transitions**: Utilizes Svelte transitions for smooth animations.
* **Icons**: Includes Svelte Hero Icons for a better user interface.

## Installation


1. Clone the repository:

   ```bash
   git clone https://github.com/pigraphic/GoogleMapsAutocomplete.git
   ```
2. Navigate to the project directory:

   ```bash
   cd googlemapsautocomplete
   ```
3. Install the dependencies:

   ```bash
   pnpm install
   ```

## Usage


1. Import the component into your Svelte application:

   ```svelte
   <script lang="ts">
       import GoogleMapsAutocomplete from './path/to/GoogleMapsAutocomplete.svelte';
   </script>
   ```
2. Use the component in your Svelte template:

   ```svelte
   <GoogleMapsAutocomplete
       label="Address"
       fieldsValues={[]}
       telephone={true}
       onchange={(name, value) => console.log(name, value)}
   />
   ```

## Props

* `label` (string): Label for the address field.
* `fieldsValues` (AutoCompleteType\[\]): Values to feed the fields.
* `telephone` (boolean): Show telephone field.
* `onchange` (function): Event handler for when the value changes.

## Types

### AutoCompleteType

```typescript
export type AutoCompleteType = {
    address: string | '';
    city: string | '';
    postal_code: string | '';
    province: string | '';
    country: string | '';
    formatted: string | '';
    telephone?: string | '';
};
```


## Contributing

Contributions are welcome! Please open an issue or submit a pull request if you have any improvements or bug fixes.

## License

This project is licensed under the MIT License. See the LICENSE file for details. \n
