<script module lang="ts">
    export type AutoCompleteType = {
        address: string | '';
        city: string | '';
        postal_code: string | '';
        province: string | '';
        country: string | '';
        formatted: string | '';
        telephone?: string | '';
    };
</script>

<script lang="ts">
    /**
     * @file GoogleMapsAutocomplete.svelte
     * @description GoogleMapsAutocomplete component using Google Maps Places API
     * @version 0.0.5
     * @Author Giulio Porta P.I.Graphic
     */
    import { PUBLIC_GOOGLE_MAPS_API_KEY } from '$env/static/public';
    import {slide} from 'svelte/transition';
    import { Icon, Plus, Trash } from 'svelte-hero-icons';
    import { onDestroy, onMount } from 'svelte';

    interface Props {
        label: string; // label for the address field
        fieldsValues?: AutoCompleteType[]; // values to feed the fields
        telephone?: boolean; // show telephone field

		// events
        onchange: (name: string, value: any) => void;
    };

    type WindowType = typeof window & {
        google: {
            maps: any;
        };
    };
    type ValuesType = Record<string, AutoCompleteType>

    const API_URL = `https://maps.googleapis.com/maps/api/js?key=${PUBLIC_GOOGLE_MAPS_API_KEY}&libraries=places`;

    // Keys must be the same as AutoCompleteType, also used to check if all keys/values are present
    const emptyValues: AutoCompleteType = {
        address: '',
        city: '',
        postal_code: '',
        province: '',
        country: '',
        formatted: '',
        telephone: '',
    };

    const prefix = 'address';
    let { onchange, label, fieldsValues, telephone, ...props }: Props = $props();

    let maps: any;
    let google: any;
    let current:number      = $state(0);
    let values:ValuesType   = $state({});
    let counter:number      = $derived( Object.keys(values).length);

    /**
     * @description Clear undefined values from object and add missing keys to be compliant with AutoCompleteType
     * @param dct: Record<string, string | undefined>
     * @returns AutoCompleteType
     */
    const clear_undefined = ( dct: Record<string, string | undefined> ):AutoCompleteType => {
        const res: any = {};
        Object.keys( emptyValues ).forEach( ( k ) => {
            if ( !dct[k] || dct[ k ] === undefined ){
                res[ k ] = '';
                 return;
            }
            res[ k ] = dct[ k ];
        } );
        return res;
    };

    /**
     * @description Load Google Maps API
     * @returns Promise
     */
    const loadGoogleMaps = () => {
        const win: WindowType = window as WindowType;
        return new Promise((resolve, reject) => {
            if (win.google && win.google.maps) {
                resolve(win.google.maps);
                return;
            }

            const script    = document.createElement('script');
            script.src      = API_URL;
            script.async    = true;
            script.defer    = true;
            script.onload   = () => resolve(win.google.maps);
            script.onerror  = reject;

            document.head.appendChild(script);
        });
    }

    /**
     * @description AutoComplete listener on address form field
     * @param e: Event
     * @returns void
     */
    const autoComplete = (e: Event) => {
        const { address_components:place } = google.getPlace();
        if (!place)
            return;

        const tmp:Record<string, string | undefined> = {};
        // clear form fields to avoid messy data when changing address
        clearFields();

        const typeMapping:Record<string, string> = {
            'route'                         : 'address',
            'street_number'                 : 'address',
            'locality'                      : 'city',
            'postal_code'                   : 'postal_code',
            'administrative_area_level_2'   : 'province',
            'country'                       : 'country'
        };
        // map google place fields to form fields and returned values
        place.map((p: any) => {
            if (!p.types) return;

            p.types.forEach((t: string) => {
                const objKey = typeMapping[t];
                if (!objKey) return;
                if (objKey) {
                    if (t === 'street_number') {
                        tmp[objKey] = p.long_name || '';
                    } else if (t === 'route') {
                        const number = tmp[objKey] === 'undefined' || tmp[objKey] === undefined ? '' : tmp[objKey];
                        tmp[objKey] = p.long_name + ' ' + number;
                    } else {
                        tmp[objKey] = p.short_name || p.long_name;
                    }
                }
            });
        });

        // add formatted address to returned values
        tmp.formatted = google.getPlace().formatted_address;
        // keep telephone value when changing address
        tmp.telephone = values[prefix+current].telephone;

        // update values so to trigger reactivity
        values[prefix+current] = { ...clear_undefined(tmp) as AutoCompleteType };
        //console.log('====> values', values);
        // call onchange event to return all values to FormCreator
        onchange (label, Object.values(values));
    }

    /**
     * @description Update FormCreator with telephone field value
     * @returns void
     */
    const telephoneField = () => {
        values[prefix+current] = { ...clear_undefined(values[prefix+current])};
        onchange (label, Object.values(values));
    }

    /**
     * @description Find targeted input field and the call addListener
     * @param e: Event
     * @returns void
     */
    const findListenerTarget = (e:Event) => {
        const input = e.target as HTMLInputElement;
        attachGoogleListener(input);
    }

    /**
     * @description Attach Google Maps Places Autocomplete listener to input field
     * @param input: HTMLInputElement
     * @returns void
     */
    const attachGoogleListener = (input:HTMLInputElement) => {
        input && input.removeEventListener(`place_changed`, autoComplete);
        google = new maps.places.Autocomplete(input, { types: ['address'], fields: ['address_components', 'formatted_address'] });
        google.addListener(`place_changed`, autoComplete);
    }

    /**
     * @description Initialize Google Maps and Places and attach listener to first input field
     * @returns void
     */
    const initPLaces = async () => {
        maps = await loadGoogleMaps();
        if (!maps || !maps.places) {
            console.error('Google Maps not loaded');
            return;
        }
    }

    /**
     * @description Feed fields with values from FormCreator
     * @returns void
     */
    const feedFields = () => {
        if (!fieldsValues || fieldsValues.length === 0) {
            addIt();
            return;
        }
        fieldsValues.forEach((f: AutoCompleteType, idx:number) => {
            addIt(idx, f);
        });
    }

    /**
     * @description Clear fields when changing address
     * @returns void
     */
    const clearFields = () => {
        let v: AutoCompleteType = { ...emptyValues };
        if (values[prefix+current].telephone !== '')
            v.telephone = values[prefix+current].telephone;
        values[prefix+current] = { ...v };
    }

    /**
     * @description Remove address item values from the values array and from the form (snippet each loop)
     * @param e: Event
     * @param id: number
     * @returns void
     */
    const trashIt = (e:Event, id:number) => {
        const input = document.getElementById(`addressField${id}`) as HTMLInputElement;
        input && input.removeEventListener(`place_changed`, autoComplete);
        if( !values[prefix+id]){
            console.warn ('Address item not found');
            return;
        }
        delete values[prefix+id];
        values = { ...values };
    }

    /**
     * @description Add a new address item to the values array and render it in the form (snippet each loop)
     * @param id: number
     * @param val: AutoCompleteType
     * @returns void
     */
    const addIt = (id?:number, val?:AutoCompleteType) => {
        id = id || counter;
        const spread = val || emptyValues;
        values[prefix+id] = { ...clear_undefined(spread) };
        current = id;
    }

    /**
     * @description Initialize Google Maps and Places and attach listener to first input field on mount
     */
    onMount (async () => {
        await initPLaces();
        feedFields();
    });

    /**
     * @description Remove all listeners on destroy
     */
    onDestroy(() => {
        for (let i = 0; i < counter; i++) {
            const input = document.getElementById(`addressField${i}`) as HTMLInputElement;
            input && input.removeEventListener(`place_changed`, autoComplete);
        }
        google = null;
    });
</script>

{#snippet trashBtn(id:number)}
    <button
        class="autocomlete-danger"
        title="Remove this address"
        onclick={(e:Event) => trashIt(e, id)}>
        <Icon src={Trash} size="1rem" />
    </button>
{/snippet}

{#snippet singleAddress(values: ValuesType, id:number)}
    <div class="col-12" transition:slide={{duration: 200}}>
        {#if telephone}
        <div class="row flex-bottom">
            <div class="col-10">
                <label for={`phoneField${id}`}>Telephone</label>
                <input
                    id={`phoneField${id}`}
                    bind:value={values[prefix+id].telephone}
                    placeholder="Enter your telephone"
                    onchange={telephoneField}
                    onfocus={() => current = id}
                />
            </div>
            <div class="offset-1 col-1">
                {@render trashBtn(id)}
            </div>
        </div>
        <div class="row">
            <div class="col-12">
                <label for={`addressField${id}`}>{label}</label>
                <input
                    id={`addressField${id}`}
                    bind:value={values[prefix+id].address}
                    placeholder="Enter your address"
                    onchange={(e:Event) => clearFields()}
                    onfocus={(e:Event) => {
                        current = id;
                        findListenerTarget(e);
                        console.log('current', current);
                    }}
                />
            </div>
        </div>
        {:else}
        <div class="row flex-bottom">
            <div class="col-10">
                <label for={`addressField${id}`}>{label}</label>
                <input
                    id={`addressField${id}`}
                    bind:value={values[prefix+id].address}
                    placeholder="Enter your address"
                    onchange={(e:Event) => clearFields()}
                    onfocus={(e:Event) => {
                        current = id;
                        findListenerTarget(e);
                        console.log('current', current);
                    }}
                />
            </div>
            <div class="offset-1 col-1">
                {@render trashBtn(id)}
            </div>
        </div>
        {/if}
        <div class="row">
            <div class="col-5">
                <input
                    bind:value={values[prefix+id].city}
                    placeholder="City"
                    disabled={true}
                />
            </div>
            <div class="col-3">
                <input
                    bind:value={values[prefix+id].postal_code}
                    placeholder="CAP"
                    disabled={true}
                />
            </div>
            <div class="col-2">
                <input
                    bind:value={values[prefix+id].province}
                    placeholder="Province"
                    disabled={true}
                />
            </div>
            <div class="col-2">
                <input
                    bind:value={values[prefix+id].country}
                    placeholder="Country"
                    disabled={true}
                />
            </div>
        </div>
    </div>
{/snippet}
<div class="row p2 y">
{#each Object.values(values) as value, idx}
    {@render singleAddress(values, idx)}
{/each}
</div>
<div class="row p2 ">
    <button
        class="autocomplete-success"
        title="Add a new address"
        onclick={() => addIt() }>
        <Icon src={Plus} size="1rem" />
    </button>
</div>