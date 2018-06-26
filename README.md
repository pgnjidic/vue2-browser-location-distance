# BrowserLocationDistance component for Vue.js

Browser location distance calculator - Vue.js component

## Install

`npm install vue2-browser-location-distance`

## Description

Vue.js component for browser location distance calculation. Works by using browser geolocation or by fall-back API call to `http://ip-api.com/json`. Also you can manually add your coordinates by parameter as well.

## Usage

First include component:

```javascript
import BrowserLocationDistance from 'vue2-browser-location-distance';

export default {
  // ...
  components: {
    BrowserLocationDistance
  }
  // ...
}

```

In JavaScript prepare locaitons that you want to calculate:

```javascript
  export default {
      //...
      data: function() {
          return {
              locations: [{
                      'name': 'Belgrade',
                      'lat': 44.7866,
                      'lon': 20.4489
                  },
                  {
                      'name': 'Zagreb',
                      'lat': 45.8150,
                      'lon': 15.9819
                  },
                  {
                      'name': 'Budapest',
                      'lat': 47.4979,
                      'lon': 19.0402
                  }
              ]
          }
      }
      //...
  }
```

Then in template usage is:

```html
<browser-location-distance :coordinates="'44.9183, 20.5586'" :order="'DESC'" :locations="locations" :unit="'K'"></browser-location-distance>
```

### Available props

| Prop  | Type  | Default  | Description  |
|---|---|---|---|
|  coordinates |  String | null  |  Manually added coordinates will disable browser or API locating. |
|  order | String  | ASC  |  Results order by *ASC* or *DESC*. |
| locations  | Array  |  null | Location array, described in text above.  |
| unit  | String  |  M |  K for kilometers (km), N for nautical miles (n.m.) and empty for miles (mi). |


### Output

```html
<table>
    <tr>
        <th>Location</th>
        <th>Distance</th>
    </tr>
    <tr>
        <td>Zagreb</td>
        <td>371.1 km</td>
    </tr>
    <tr>
        <td>Budapest</td>
        <td>309.69 km</td>
    </tr>
    <tr>
        <td>Belgrade</td>
        <td>17.01 km</td>
    </tr>
</table>
```
