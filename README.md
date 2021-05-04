# WIP

## Usage

```svelte
<Mapkit
    jwt={"YOUR_JWT"}
    region={region}
    on:ready={(e) => {
      console.log(e.detail.map,e.detail.mapkit)
    }}
    on:region-change={(e) => {
      console.log(e.detail.region)
    }}
    options={{}}
  />
```
