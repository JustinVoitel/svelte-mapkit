<script lang="ts">
  import loader from "@beyonk/async-script-loader";
  import { onMount } from "svelte";
  import { createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();
  export let jwt: any;
  export let version = "5.45.0";
  export let language = "de";
  export let options: mapkit.MapConstructorOptions = {};
  export let region: {
    center: { latitude: number; longitude: number };
    span: { latitudeDelta: number; longitudeDelta: number };
  } = {
    center: { latitude: 52.52097071474431, longitude: 13.411468164580242 },
    span: { latitudeDelta: 0.0126, longitudeDelta: 0.0418 },
  };

  let map: mapkit.Map = null;
  let mapkit: typeof window.mapkit;
  let container;

  function onAvailable(alreadyInitialized: boolean) {
    mapkit = window.mapkit;
    const optionsWithDefaults = Object.assign(
      {
        //visible portion of map
        region: new mapkit.CoordinateRegion(
          new mapkit.Coordinate(
            region.center.latitude,
            region.center.longitude
          ),
          new mapkit.CoordinateSpan(
            region.span.latitudeDelta,
            region.span.longitudeDelta
          )
        ),
        //map appearance and controls
        showsCompass: mapkit.FeatureVisibility.Visible,
        showsZoomControl: true,
        showsMapTypeControl: true,
        showsScale: mapkit.FeatureVisibility.Visible,
        colorScheme: "dark", //mapkit.Map.ColorSchemes.Dark
        // tintColor: 'var(--primary-300)',
        //user location
        showsUserLocation: false,
        showsUserLocationControl: false,
        tracksUserLocation: false,
        //other
      },
      options
    );
    if (!alreadyInitialized) {
      const initOptions = {
        authorizationCallback: function (done) {
          done(jwt);
        },
        language,
      };
      mapkit.init(initOptions);
    }
    let el = new mapkit.Map(container, optionsWithDefaults);
    map = el;
    // store._initMap(map, mapkit)

    // el.addEventListener('zoom-start', (e) => {
    //   // console.log('zoom-start')
    // })
    // el.addEventListener('zoom-end', (e) => {
    //   // console.log('zoom-end', e.target.region)
    //   dispatch("zoom-change",{
    //     span: e.target.
    //   })
    //   store?.update((u) => ({ ...u, span: e.target.region.span }))
    // })

    el.addEventListener("region-change-start", (e) => {
      // console.log('region-change-start', e)
    });
    el.addEventListener("region-change-end", (e) => {
      // store?.update((u) => ({ ...u, center: e.target.center }))
      dispatch("region-change", {
        region: e.target.region,
      });
    });

    dispatch("ready", {
      map,
      mapkit,
    });
  }

  onMount(() => {
    const resources = [
      {
        type: "script",
        url: `//cdn.apple-mapkit.com/mk/${version}/mapkit.js`,
      },
    ];
    loader(
      resources,
      () => window.mapkit,
      () => onAvailable(false)
    );
    return () => {
      if (map && map.destroy) {
        map.destroy();
      }
    };
  });
</script>

<div class="h-full w-full mapkit" bind:this={container}>
  {#if map}
    <slot />
  {/if}
</div>
