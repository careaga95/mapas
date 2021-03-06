---
published: true
category: maps
layout: map
title: "Huracán: Manuel"
splash: "manuel.png"
basemap: "devseed.jfkl2aak"
endpoints:
 - url: "http://catalogo.datos.gob.mx/api/action/datastore_search?resource_id=acbc6f7f-32cd-467d-8ee0-a71b4db7d647&fields=CLAVE,Date,EVENTO&sort=Date desc"
   title: "Reconstruction"
dropdown-menu: "http://catalogo.datos.gob.mx/api/action/datastore_search?resource_id=acbc6f7f-32cd-467d-8ee0-a71b4db7d647&fields=Code,Date,EVENTO&sort=Date desc&limit=100000"
info: "El Fondo de Desastres Naturales de México (FONDEN) fue establecido a finales de los años 90’s como un mecanismo presupuestario para apoyar de manera eficaz y oportuna a la rehabilitación de la infraestructura federal y estatal afectada por desastres naturales. En la actualidad, el FONDEN está compuesto por dos instrumentos presupuestarios complementarios: el Programa FONDEN para la Reconstrucción y el Programa Fondo para la Prevención de Desastres Naturales (FOPREDEN), y sus respectivos fideicomisos."
custom: true
tour: 
 - title: "Manual huracán golpea la costa oeste de México en <b>Septiembre 18, 2013</b>."
   lat: "0"
   lng: "0"
   zoom: "7"
 - title: "Municiaplities en el <b>estado de Sinaloa</b> sienten el impacto de la tormenta.</b>"
   lat: "0"
   lng: "0"
   zoom: "8"
   layer: "municipalitiesLayer"
 - title: "Daños en viviendas y infastruture son significativas y los resultados en un número sifnifcant de los <b>proyectos de reconstrucción.</b>"
   lat: "0"
   lng: "0"
   zoom: "9"
   layer: "markerLayer"
---

<div id="explorer" class="container-fluid">
<div class="row">
   <div class="event-info col-md-12">
      Date: Event / Municipality / State
   </div>
   <div class="explorer-label col-xs-3 col-sm-3 col-md-3">
     <h2>Emergencia</h2>
   </div>
   <div class="explorer-label col-xs-3 col-sm-3 col-md-3">
     <h2>APIN</h2>
   </div>
   <div class="explorer-label col-xs-6 col-sm-6 ol-md-6">
     <h2>Reconstrucci&oacute;n</h2>
   </div>
   <div class="data-wrapper emergency-wrapper col-xs-3 col-sm-3 col-md-3">
     <table id="emergency" class="table"></table>
  </div>
  <div class="data-wrapper apin-wrapper col-xs-3 col-sm-3 col-md-3">
    <table id="apin" class="table"></table>   
  </div>
  <div class="data-wrapper projects-wrapper col-xs-6 col-sm-6 col-md-6">
    <table id="projects" class="table"></table>
  </div>
</div>
</div>

<!-- Modal -->
<div class="modal fade" id="imageModal" tabindex="-1" role="dialog" aria-labelledby="myModalLabel" aria-hidden="true">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <button type="button" class="close" data-dismiss="modal"><span aria-hidden="true">&times;</span><span class="sr-only">Close</span></button>
        <h4 class="modal-title" id="myModalLabel">Title</h4>
      </div>
      <div class="modal-body">
        <img class="modal-image" src="#" />
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
      </div>
    </div>
  </div>
</div>


<ul class="tour">
{% for item in page.tour %}
<li class="tour-item" data-layer="{{item.layer}}" data-zoom="{{item.zoom}}" data-lat="{{item.lat}}" data-lng="{{item.lng}}">{{item.title}}</li>
{% endfor %}
</ul>
<a class="tour-nav tour-next-btn" href="#">></a>




<script type='text/javascript'>

$('.loading').show();
$('#info').hide();
$('#explorer').hide();
$('#pager').hide();


L.mapbox.accessToken = 'pk.eyJ1IjoiZGV2c2VlZCIsImEiOiJnUi1mbkVvIn0.018aLhX0Mb0tdtaT2QNe2Q';
var map = L.mapbox.map('map', null).setView([24.6292882, -102.7022955], 5);
var baseLayer = L.mapbox.tileLayer('{{page.basemap}}').addTo(map);
var overlayLayers = L.layerGroup().addTo(map);
var municipalitiesLayer = L.geoJson();
var markerLayer = new L.MarkerClusterGroup({ showCoverageOnHover: true, zoomToBoundsOnClick: true, disableClusteringAtZoom: 13, removeOutsideVisibleBounds: true});
var municipalityKey = [];


console.log(markerLayer);

var icon = {
    "iconUrl": '{{site.baseurl}}/css/images/icon.png',
    "iconSize": [20, 20],
    "opacity": 0.2
};

new L.Control.MiniMap(L.mapbox.tileLayer('{{page.basemap}}'), {
        aimingRectOptions: {
            color: '#FF0000'
        }
    })
    //.addTo(map);

function countryStyle(feature) {
    return {
        'weight': 2,
        'opacity': 0.9,
        'color': '#FF0000',
        'fillOpacity': 0.6,
        'fillColor': '#FF0000'
    }
}

function countryStyleHover(feature) {
    return {
        'weight': 2,
        'opacity': 0.9,
        'color': '#FF0000',
        'fillOpacity': 0.6,
        'fillColor': '#8C0000'
    }
}



init();


function init() {

    var datos = [];

    $('.loading').show();
    $('#explorer').hide();
    $('.layer-switch li a').removeClass('active');
    $(this).addClass('active');
    municipalitiesLayer.clearLayers();
    markerLayer.clearLayers();
    var query = $(this).data('id');
    var event = $(this).text();
    $('#event-select-label').text(event);
    $('.tour-item').hide();


	$('#projects tr').remove();
	$('#projects').append('<tr><td>Proyecto</td><td>Importe</td></tr>');


    $.ajax({
        type: 'GET',
        url: 'http://catalogo.datos.gob.mx/api/action/datastore_search?resource_id=acbc6f7f-32cd-467d-8ee0-a71b4db7d647&filters={"Code": "20100180920130125"}&limit=100000',
        dataType: 'jsonp',
        cache: true,
        success: function(data) {

            //console.log(data);

            $.each(data.result.records, function(index, value) {

                if (value.MunId) {

                    var munId = value.MunId.toString();
                    if (munId.length == 4) {
                        munId = '0' + munId;
                    }
                    var stateId = munId.substring(0, 2);
                    var munId = munId.substring(2);

					
					if (value['_id'] == 6037 || value['_id'] == 5637 || value['_id'] == 4490) {
					  // leave out these outliers
					} else {
						var marker = L.marker(new L.LatLng(value['LATITUD'], value['LONGITUD']), {
							'id': 'record-' + value['_id'],
                        	'eventId':  value['CLAVE']
						});
					
						marker.setIcon(L.icon(icon));
						markerLayer.addLayer(marker);
                    }

                    if (value['ACCION'] != null) {
                        $('#projects').append('<tr class="project record-' + value['_id'] + '"><td class="record-description"><p>' + value['ACCION'] + '</p></td><td>' + withCommas(value['MONTO.RECONSTRUCCION']) + '</td></tr>');
                    }

                    datos.push({
                        'stateId': stateId,
                        'munId': munId,
                        'record': value
                    });


                }

            });


            var activeMuns = [];
            $.each(datos, function(index, value) {
                var mid = value.munId + value.stateId;
                var check = $.inArray(mid, activeMuns);
                if (check == -1) {
                    L.geoJson(municipalities, {
                        style: countryStyle,
                        filter: function(feature, layer) {
                            return feature.properties['CVE_MUN'] == value.munId && feature.properties['CVE_ENT'] == value.stateId;
                        }
                    }).addTo(municipalitiesLayer);
                    activeMuns.push(mid);
                }
            });




            $('.loading').hide();

            $('.tour-item:first-child').show();
            $('.tour-nav').show();
            var tourCount = 0;

            $('.tour-next-btn').click(function(e) {
                e.preventDefault();
                tourCount++;

                var tourLength = $('ul.tour').length;
                var tourItems = $('ul.tour li');

                if (tourCount > tourLength + 1) {
                    $('.tour-item, .tour-nav').fadeOut();
                }

                if (tourCount == 0) {
                    overlayLayers.clearLayers();
                    $('#explorer').hide();
                }
                
                var tourItemZoom = $(tourItems[tourCount]).data('zoom');
                var tourItemLayer = $(tourItems[tourCount]).data('layer');

                $('.tour-item').hide();
                $(tourItems[tourCount]).show();

                if (tourItemLayer == 'markerLayer') {
                    markerLayer.addTo(overlayLayers);
                }
                if (tourItemLayer == 'municipalitiesLayer') {
                    municipalitiesLayer.addTo(overlayLayers);
                }
                map.setZoom(tourItemZoom);


            });



            map.fitBounds(municipalitiesLayer.getBounds(), {
                maxZoom: 9
            });


        }

    });



    $.ajax({
        type: 'GET',
        url: 'http://201.175.32.249/api/action/datastore_search?resource_id=39e78078-495e-4c0e-a202-4b6668a226b9&filters={"Code": "20100180920130125"}&limit=100000',
        dataType: 'jsonp',
        cache: true,
        success: function(data) {

            $.each(data.result.records, function(index, value) {

                if (value.MunId) {

                    var munId = value.MunId.toString();
                    if (munId.length == 4) {
                        munId = '0' + munId;
                    }

                    var stateId = munId.substring(0, 2);
                    var munId = munId.substring(2);


                    datos.push({
                        'stateId': stateId,
                        'munId': munId,
                        'record': value
                    });

                }

            });

        }

    });


    $.ajax({
        type: 'GET',
        url: 'http://201.175.32.249/api/action/datastore_search?resource_id=37814da6-8d53-4287-a8c4-86492a636cfb&filters={"Code": "20100180920130125"}&limit=100000',
        dataType: 'jsonp',
        cache: true,
        success: function(data) {

            $.each(data.result.records, function(index, value) {

                if (value.MunId) {

                    var munId = value.MunId.toString();
                    if (munId.length == 4) {
                        munId = '0' + munId;
                    }

                    var stateId = munId.substring(0, 2);
                    var munId = munId.substring(2);


                    datos.push({
                        'stateId': stateId,
                        'munId': munId,
                        'record': value
                    });

                }

            });
            
            
            municipalitiesLayer.on('click', function(o) {
                if (o) {

                    $('#explorer').fadeIn();
                    $('#emergency tr').remove();
                    $('#apin tr').remove();
                    
                    $('#emergency').append('<tr><td>Articulo</td><td>Importe</td></tr>');
                    $('#apin').append('<tr><td>Articulo</td><td>Federal</td><td>Estatal</td></tr>');

                    $.each(datos, function(index, value) {
                        if (value.munId === o.layer.feature.properties.CVE_MUN) {
                            $('.event-info').empty().append(value.record['EVENTO'] + ': ' + value.record['Date'] + ' / ' + value.record['ESTADO'] + ' / ' + value.record['MUNICIPIO']);
                            if (value.record.Despensas != null) {
                                $('#emergency').append('<tr><td>Despensas</td>' + '<td>' + withCommas(value.record['Despensas']) + '</td></tr>' + '<tr><td>Cobertores</td>' + '<td>' + withCommas(value.record['Cobertores']) + '</td></tr>' + '<tr><td>Colchenetas</td>' + '<td>' + withCommas(value.record['Colchonetas']) + '</td></tr>' + '<tr><td>Impermeables</td>' + '<td>' + withCommas(value.record['Impermeables']) + '</td></tr>' + '<tr><td>Guantes de Carnaza</td>' + '<td>' + withCommas(value.record['Guantes.de.Carnaza']) + '</td></tr>' + '<tr><td>Rollos de Hule</td>' + '<td>' + withCommas(value.record['Rollos.de.Hule']) + '</td></tr>' + '<tr><td>Lamina Tipo B</td>' + '<td>' + withCommas(value.record['Lámina.Tipo.B']) + '</td></tr>' + '<tr><td>Botas</td>' + '<td>' + withCommas(value.record['Botas']) + '</td></tr>' + '<tr><td>Kits de Aseo.Personal</td>' + '<td>' + withCommas(value.record['Kits.de.Aseo.Personal']) + '</td></tr>' + '<tr><td>Kits de Limpieza</td>' + '<td>' + withCommas(value.record['Kits.de.Limpieza']) + '</td></tr>' + '<tr><td>Litros de Agua</td>' + '<td>' + withCommas(value.record['Litros.de.Agua']) + '</td></tr>' + '<tr><td>Costales</td>' + '<td>' + withCommas(value.record['Costales']) + '</td></tr>' + '<tr><td>Linternas</td>' + '<td>' + withCommas(value.record['Linternas']) + '</td></tr>' + '<tr><td>Toallas.Sanitarias</td>' + '<td>' + withCommas(value.record['Toallas.Sanitarias']) + '</td></tr>' + '<tr><td>Panales para Bebe</td>' + '<td>' + withCommas(value.record['Pañales.para.Bebé']) + '</td></tr>' + '<tr><td>Panal para Adulto</td>' + '<td>' + withCommas(value.record['Pañal.para.Adulto']) + '</td></tr>' + '<tr><td>Marros</td>' + '<td>' + withCommas(value.record['Marros']) + '</td></tr>' + '<tr><td>Barretas</td>' + '<td>' + withCommas(value.record['Barretas']) + '</td></tr>' + '<tr><td>Carretillas</td>' + '<td>' + withCommas(value.record['Carretillas']) + '</td></tr>' + '<tr><td>Palas</td>' + '<td>' + withCommas(value.record['Palas']) + '</td></tr>' + '<tr><td>Zapapicos</td>' + '<td>' + withCommas(value.record['Zapapicos']) + '</td></tr>' + '<tr><td>Hachas</td>' + '<td>' + withCommas(value.record['Hachas']) + '</td></tr>' + '<tr><td>Machetes</td>' + '<td>' + withCommas(value.record['Machetes']) + '</td></tr>' + '<tr><td>Martillos</td>' + '<td>' + withCommas(value.record['Martillos']) + '</td></tr>' + '<tr><td>Azadones</td>' + '<td>' + withCommas(value.record['Azadones']) + '</td></tr>' + '<tr><td>Cinceles</td>' + '<td>' + withCommas(value.record['Cinceles']) + '</td></tr>' + '<tr><td>Cascos</td>' + '<td>' + withCommas(value.record['Cascos']) + '</td></tr>' + '<tr><td>Combustible</td>' + '<td>' + withCommas(value.record['Combustible']) + '</td></tr>' + '<tr><td>Guantes de Neopreno</td>' + '<td>' + withCommas(value.record['Guantes.de.Neopreno']) + '</td></tr>' + '<tr><td>Lamina Tipo A</td>' + '<td>' + withCommas(value.record['Lámina.Tipo.A']) + '</td></tr>' + '<tr><td>Lamina Tipo C</td>' + '<td>' + withCommas(value.record['Lámina.Tipo.C']) + '</td></tr>' + '<tr><td>Bolsa para Cadaver</td>' + '<td>' + withCommas(value.record['Bolsa.para.Cadáver']) + '</td></tr>');
                            }
                        }
                    });
                    $.each(datos, function(index, value) {
                        if (value.munId === o.layer.feature.properties.CVE_MUN) {
                            if (value.record.Sector != null) {
                                $('#apin').append('<tr><td>' + value.record['Sector'] + '</td><td>' + value.record['Infraestructura.federal'] + '</td><td>' + value.record['Infraestructura.estatal'] + '</td></tr>');
                            }
                        }
                    });
                    
                    
                    if ($('#emergency tr').length == 1) {
                        $('#emergency').append('<tr><td>No hay datos para este evento.</td><td>-</td></tr>');
                    }

                    if ($('#apin tr').length == 1) {
                        $('#apin').append('<tr><td>No hay datos para este evento.</td><td>-</td><td>-</td></tr>');
                    }

                }

            });



            municipalitiesLayer.on('mousemove', function(o) {
                if (o) {

                    $('#explorer').fadeIn();
                    $('#emergency tr').remove();
                    $('#apin tr').remove();
                    
                    $('#emergency').append('<tr><td>Articulo</td><td>Importe</td></tr>');
                    $('#apin').append('<tr><td>Articulo</td><td>Federal</td><td>Estatal</td></tr>');


                    $.each(datos, function(index, value) {
                        if (value.munId === o.layer.feature.properties.CVE_MUN) {
                            $('.event-info').empty().append(value.record['EVENTO'] + ': ' + value.record['Date'] + ' / ' + value.record['ESTADO'] + ' / ' + value.record['MUNICIPIO']);
                            if (value.record.Despensas != null) {
                                $('#emergency').append('<tr><td>Despensas</td>' + '<td>' + withCommas(value.record['Despensas']) + '</td></tr>' + '<tr><td>Cobertores</td>' + '<td>' + withCommas(value.record['Cobertores']) + '</td></tr>' + '<tr><td>Colchenetas</td>' + '<td>' + withCommas(value.record['Colchonetas']) + '</td></tr>' + '<tr><td>Impermeables</td>' + '<td>' + withCommas(value.record['Impermeables']) + '</td></tr>' + '<tr><td>Guantes de Carnaza</td>' + '<td>' + withCommas(value.record['Guantes.de.Carnaza']) + '</td></tr>' + '<tr><td>Rollos de Hule</td>' + '<td>' + withCommas(value.record['Rollos.de.Hule']) + '</td></tr>' + '<tr><td>Lamina Tipo B</td>' + '<td>' + withCommas(value.record['Lámina.Tipo.B']) + '</td></tr>' + '<tr><td>Botas</td>' + '<td>' + withCommas(value.record['Botas']) + '</td></tr>' + '<tr><td>Kits de Aseo.Personal</td>' + '<td>' + withCommas(value.record['Kits.de.Aseo.Personal']) + '</td></tr>' + '<tr><td>Kits de Limpieza</td>' + '<td>' + withCommas(value.record['Kits.de.Limpieza']) + '</td></tr>' + '<tr><td>Litros de Agua</td>' + '<td>' + withCommas(value.record['Litros.de.Agua']) + '</td></tr>' + '<tr><td>Costales</td>' + '<td>' + withCommas(value.record['Costales']) + '</td></tr>' + '<tr><td>Linternas</td>' + '<td>' + withCommas(value.record['Linternas']) + '</td></tr>' + '<tr><td>Toallas.Sanitarias</td>' + '<td>' + withCommas(value.record['Toallas.Sanitarias']) + '</td></tr>' + '<tr><td>Panales para Bebe</td>' + '<td>' + withCommas(value.record['Pañales.para.Bebé']) + '</td></tr>' + '<tr><td>Panal para Adulto</td>' + '<td>' + withCommas(value.record['Pañal.para.Adulto']) + '</td></tr>' + '<tr><td>Marros</td>' + '<td>' + withCommas(value.record['Marros']) + '</td></tr>' + '<tr><td>Barretas</td>' + '<td>' + withCommas(value.record['Barretas']) + '</td></tr>' + '<tr><td>Carretillas</td>' + '<td>' + withCommas(value.record['Carretillas']) + '</td></tr>' + '<tr><td>Palas</td>' + '<td>' + withCommas(value.record['Palas']) + '</td></tr>' + '<tr><td>Zapapicos</td>' + '<td>' + withCommas(value.record['Zapapicos']) + '</td></tr>' + '<tr><td>Hachas</td>' + '<td>' + withCommas(value.record['Hachas']) + '</td></tr>' + '<tr><td>Machetes</td>' + '<td>' + withCommas(value.record['Machetes']) + '</td></tr>' + '<tr><td>Martillos</td>' + '<td>' + withCommas(value.record['Martillos']) + '</td></tr>' + '<tr><td>Azadones</td>' + '<td>' + withCommas(value.record['Azadones']) + '</td></tr>' + '<tr><td>Cinceles</td>' + '<td>' + withCommas(value.record['Cinceles']) + '</td></tr>' + '<tr><td>Cascos</td>' + '<td>' + withCommas(value.record['Cascos']) + '</td></tr>' + '<tr><td>Combustible</td>' + '<td>' + withCommas(value.record['Combustible']) + '</td></tr>' + '<tr><td>Guantes de Neopreno</td>' + '<td>' + withCommas(value.record['Guantes.de.Neopreno']) + '</td></tr>' + '<tr><td>Lamina Tipo A</td>' + '<td>' + withCommas(value.record['Lámina.Tipo.A']) + '</td></tr>' + '<tr><td>Lamina Tipo C</td>' + '<td>' + withCommas(value.record['Lámina.Tipo.C']) + '</td></tr>' + '<tr><td>Bolsa para Cadaver</td>' + '<td>' + withCommas(value.record['Bolsa.para.Cadáver']) + '</td></tr>');
                            }
                        }
                    });
                    $.each(datos, function(index, value) {
                        if (value.munId === o.layer.feature.properties.CVE_MUN) {
                            if (value.record.Sector != null) {
                                $('#apin').append('<tr><td>' + value.record['Sector'] + '</td><td>' + value.record['Infraestructura.federal'] + '</td><td>' + value.record['Infraestructura.estatal'] + '</td></tr>');
                            }
                        }
                    });


                    if ($('#emergency tr').length == 1) {
                        $('#emergency').append('<tr><td>No hay datos para este evento.</td><td>-</td></tr>');
                    }

                    if ($('#apin tr').length == 1) {
                        $('#apin').append('<tr><td>No hay datos para este evento.</td><td>-</td><td>-</td></tr>');
                    }


                }

            });
            
        }

    });


}

/*
markerLayer.on('clusterclick', function(e) {
  e.layer.spiderfy();
});
*/

markerLayer.on('click', function(e) {
    console.log(e.layer.options.id);
    var targetRecord = '.' + e.layer.options.id;
    var eventId = e.layer.options.eventId;
    $('.project').css('background', 'none');
    $('.project').css('color', '#777');
    $(targetRecord).css('background', '#ccc');
    $(targetRecord).css('color', '#000');
    $(targetRecord).ScrollTo();
    if (! $(targetRecord).hasClass('images-added')) {
    $(targetRecord + ' td.record-description').append('<a href="#" class="modal-trigger" data-toggle="modal" data-target="#imageModal" data-image="https://s3.amazonaws.com/fondephotos/Fotos-out/' + eventId + '-a.jpg"><img onerror="imgError(this);" class="project-image" src="https://s3.amazonaws.com/fondephotos/Fotos-out/' + eventId + '-a.jpg" /></a>'
    					+ '<a href="#" class="modal-trigger" data-toggle="modal" data-target="#imageModal" data-image="https://s3.amazonaws.com/fondephotos/Fotos-out/' + eventId + '-b.jpg"><img onerror="imgError(this);" class="project-image" src="https://s3.amazonaws.com/fondephotos/Fotos-out/' + eventId + '-b.jpg" /></a>' 
    					+ '<a href="#" class="modal-trigger" data-toggle="modal" data-target="#imageModal" data-image="https://s3.amazonaws.com/fondephotos/Fotos-out/' + eventId + '-c.jpg"><img onerror="imgError(this);" class="project-image" src="https://s3.amazonaws.com/fondephotos/Fotos-out/' + eventId + '-c.jpg" /></a>'
    					+ '<a href="#" class="modal-trigger" data-toggle="modal" data-target="#imageModal" data-image="https://s3.amazonaws.com/fondephotos/Fotos-out/' + eventId + '-d.jpg"><img onerror="imgError(this);" class="project-image" src="https://s3.amazonaws.com/fondephotos/Fotos-out/' + eventId + '-d.jpg" /></a>');
    } 
    $(targetRecord).addClass('images-added');
});


$('body').on('click', '.modal-trigger', function (){
    var image = $(this).data('image'); 
    $('.modal-image').attr('src', image);
});


/*
municipalitiesLayer.on('mouseover', function(e) {
    var layer = e.layer,
        feature = layer.feature;
    layer.setStyle(countryStyleHover(layer));
});


municipalitiesLayer.on('mouseout', function(e) {
    var layer = e.layer,
        feature = layer.feature;
    layer.setStyle(countryStyle(layer));
});
*/

var hash = window.location.hash;
if (hash === '#embed') {
    $('body').addClass('embed');
    map.scrollWheelZoom.disable();
}


function withCommas(x) {
    return x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
}

function formatDate(x) {
    var l = x.toString().length;
    if (l === 7) {
        x = '0' + x;
    }
    var d = x.substring(0, 2);
    var m = x.substring(2, 4);
    var y = x.substring(4);
    return m + '/' + d + '/' + y;
}

function imgError(image){
    $(image).hide();
}


</script>
