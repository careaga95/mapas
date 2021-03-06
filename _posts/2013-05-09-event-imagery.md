---
published: true
category: maps
layout: map
title: "2013/09/20: LLUVIA SEVERA"
splash: "event-0915.png"
basemap: "devseed.fgayuacn"
lat: "22.12"
lng: "-98.05"
zoom: "11"
minZoom: "10"
maxZoom: "13"
info: "El 20 de septiembre de 2013 una serie de lluvias torrenciales, provocadas por el huracán Ingrid, causaron serios estragos en toma la costa del Golfo de México. En particular la ciudad de Tampico, Tamaulipas se vio fuertemente afectada y después de evento y requirió de más de 40 proyectos de reconstrucción de infraestructura financiados por el Fondo de Desastres Naturales. En la siguiente herramienta usando el slider en la parte superior, usando imágenes satelitales infrarrojas, puedes observar como se veía Tampico antes (lado derecho) y después del huracán ingrid. En particular es notable el incremento del tamaño de los cuerpos de agua en la región. Adicionalmente se incluye un marcador en la ubicación de cada uno de los proyectos de reconstrucción posteriores."
---

<div id="slider">
<span class="layer-label label-left">Nivel de agua normal</span>
<span class="layer-label label-right">20 de Septiembre 2013</span>
</div>

<a target="_blank" href="#" id="fullscreen">
    <svg xmlns="http://www.w3.org/2000/svg" version="1.1" x="0" y="0" width="100%" height="100%" viewBox="0 0 512 512" enable-background="new 0 0 512 512" xml:space="preserve"><path style="fill: #fff;" d="M157.9 426.9L192.9 462H50V319.1l35.1 35 57.3-57.3 72.9 72.9L157.9 426.9zM319.1 50l35 35.1 -56.1 56.1 72.9 72.9 56.1-56.1L462 192.9V50H319.1zM85.1 157.9L50 192.9V50h142.9L157.9 85.1l57.3 57.3 -72.9 72.9L85.1 157.9zM462 319.1l-35.1 35 -56.1-56.1 -72.9 72.9 56.1 56.1L319.1 462H462V319.1z"/></svg>
</a>

<script type='text/javascript'>

$('#slider').append("<input id='range' class='range' type='range' min='0' max='1.0' step='any' />");

$('.loading').show();
$('#info').hide();
$('#pager').hide();

L.mapbox.accessToken = 'pk.eyJ1IjoiZGV2c2VlZCIsImEiOiJnUi1mbkVvIn0.018aLhX0Mb0tdtaT2QNe2Q';
var map = L.mapbox.map('map', null, { maxZoom: {{page.maxZoom}}, minZoom: {{page.minZoom}} }).setView([{{page.lat}}, {{page.lng}}], {{page.zoom}});
var baseLayer = L.mapbox.tileLayer('devseed.tampico2014');
map.addLayer(baseLayer);
var municipalitesLayer = L.geoJson();
var overlay = L.mapbox.tileLayer('devseed.tampicoFlooded').addTo(map);
var range = document.getElementById('range');
var markers = L.mapbox.featureLayer();
var municipalityKey = [];
municipalitesLayer.addTo(map);
markers.addTo(map);


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
        cache: true,
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
                            'id': munId
                        });

                      var markup = '<img onerror="imgError(this);" class="project-image-tooltip" src="https://s3.amazonaws.com/fondephotos/Fotos-out/' + value['CLAVE'] + '-a.jpg" /><b>' + value['MUNICIPIO'] + '</b><br><small>' + value['ESTADO'] + '</small><br>' + value['TIPO.DE.APOYO'] + ', ' + withCommas(value['MONTO.RECONSTRUCCION'] + ' MXP');


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
    map.scrollWheelZoom.disable();
}
   
         
function withCommas(x) {
    return x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
}


function clip() {
  var nw = map.containerPointToLayerPoint([0, 0]),
      se = map.containerPointToLayerPoint(map.getSize()),
      clipX = nw.x + (se.x - nw.x) * range.value;

  overlay.getContainer().style.clip = 'rect(' + [nw.y, clipX, se.y, nw.x].join('px,') + 'px)';
}

range['oninput' in range ? 'oninput' : 'onchange'] = clip;
map.on('move', clip);

clip();

function imgError(image){
    $(image).hide();
}


</script>  