---
published: false
category: maps
layout: map
title: "2013/09/20: LLUVIA SEVERA"
splash: "event-0920.png"
basemap: "devseed.fgayuacn"
lat: "22.2706519"
lng: "-97.850"
zoom: "13"
---

<script type='text/javascript'>

$('.loading').show();

L.mapbox.accessToken = 'pk.eyJ1IjoiZGV2c2VlZCIsImEiOiJnUi1mbkVvIn0.018aLhX0Mb0tdtaT2QNe2Q';
var map = L.mapbox.map('map', null).setView([{{page.lat}}, {{page.lng}}], {{page.zoom}});
var baseLayer = L.mapbox.tileLayer('devseed.tampico');
map.addLayer(baseLayer);
baseLayer.setOpacity(0.8);
var municipalitesLayer = L.geoJson();
var apiLayer = L.layerGroup();
var markers = L.mapbox.featureLayer();
var municipalityKey = [];
municipalitesLayer.addTo(map);
markers.addTo(map);
apiLayer.addTo(map);

new L.Control.MiniMap(L.mapbox.tileLayer('devseed.jfe5nhb2'), {
        aimingRectOptions: {
            color: '#FFBF00'
        }
    })
    .addTo(map);

$.ajax({
    type: 'GET',
    url: 'http://catalogo.datos.gob.mx/api/action/datastore_search?resource_id=acbc6f7f-32cd-467d-8ee0-a71b4db7d647&filters={"Code": "20300200920130130"}&limit=100000',
    dataType: 'jsonp',
    success: function(data) {

        console.log(data);

        $.each(data.result.records, function(index, value) {

            if (value.MunId) {

                var munId = value.MunId.toString();
                if (munId.length == 4) {
                    munId = '0' + munId;
                }
                var stateId = munId.substring(0, 2);
                var munId = munId.substring(2);


                var marker = L.marker(new L.LatLng(value['LATITUD'], value['LONGITUD']), {
                    'title': 'test',
                    'id': munId
                });

                var markup = value['TIPO.DE.APOYO'];

                // marker.setIcon(L.icon(icon));
                marker.bindPopup(markup, {
                    autoPan: true
                });

                markers.addLayer(marker);

                $('.loading').hide();


            }

        });

    }
});


var hash = window.location.hash;
if (hash === '#embed') {
    $('body').addClass('embed');
}


</script>
