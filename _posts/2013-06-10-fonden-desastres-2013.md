---
published: true
category: maps
layout: map
title: "Fondo de Desastres Naturales 2013"
splash: "map-bg-1.png"
basemap: "devseed.jfkl2aak"
endpoints:
 - url: "http://catalogo.datos.gob.mx/api/action/datastore_search?resource_id=acbc6f7f-32cd-467d-8ee0-a71b4db7d647&fields=CLAVE,Date,EVENTO&sort=Date desc"
   title: "Reconstruction"
dropdown-menu: "http://catalogo.datos.gob.mx/api/action/datastore_search?resource_id=acbc6f7f-32cd-467d-8ee0-a71b4db7d647&fields=Code,Date,EVENTO&sort=Date desc&limit=100000"
info: "El Fondo de Desastres Naturales de México (FONDEN) fue establecido a finales de los años 90’s como un mecanismo presupuestario para apoyar de manera eficaz y oportuna a la rehabilitación de la infraestructura federal y estatal afectada por desastres naturales. En la actualidad, el FONDEN está compuesto por dos instrumentos presupuestarios complementarios: el Programa FONDEN para la Reconstrucción y el Programa Fondo para la Prevención de Desastres Naturales (FOPREDEN), y sus respectivos fideicomisos."
custom: true
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

<a target="_blank" href="#" id="fullscreen">
    <svg xmlns="http://www.w3.org/2000/svg" version="1.1" x="0" y="0" width="100%" height="100%" viewBox="0 0 512 512" enable-background="new 0 0 512 512" xml:space="preserve"><path style="fill: #fff;" d="M157.9 426.9L192.9 462H50V319.1l35.1 35 57.3-57.3 72.9 72.9L157.9 426.9zM319.1 50l35 35.1 -56.1 56.1 72.9 72.9 56.1-56.1L462 192.9V50H319.1zM85.1 157.9L50 192.9V50h142.9L157.9 85.1l57.3 57.3 -72.9 72.9L85.1 157.9zM462 319.1l-35.1 35 -56.1-56.1 -72.9 72.9 56.1 56.1L319.1 462H462V319.1z"/></svg>
</a>

<script type='text/javascript'>


$('.loading').show();
$('#info').hide();
$('#explorer').hide();
$('#pager').hide();


L.mapbox.accessToken = 'pk.eyJ1IjoiZGV2c2VlZCIsImEiOiJnUi1mbkVvIn0.018aLhX0Mb0tdtaT2QNe2Q';
var map = L.mapbox.map('map', '{{page.basemap}}').setView([24.6292882, -102.7022955], 5);
var municipalitiesLayer = L.geoJson();
var markerLayer = new L.MarkerClusterGroup({ showCoverageOnHover: true, zoomToBoundsOnClick: true, disableClusteringAtZoom: 13, removeOutsideVisibleBounds: true});
var municipalityKey = [];
municipalitiesLayer.addTo(map);
markerLayer.addTo(map);


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



$.ajax({
    type: 'GET',
    url: '{{page.dropdown-menu}}',
    dataType: 'jsonp',
    cache: true,
    success: function(data) {

        var itemsUniqueKey = [];
        var items = [];

        $.each(data.result.records, function(index, value) {
            var check = $.inArray(value.Code, itemsUniqueKey);
            if (check == -1) {
                itemsUniqueKey.push(value.Code);
                items.push(value);
            }
        });


        var i = items.length;
        var itemWidth = 100 / i;

        $.each(items, function(index, value) {
            $('.date-dropdown').append('<li><a data-id="' + value.Code + '" href="#">' + value['Date'] + ': ' + value['EVENTO'] + '</a></li>');
            $('#pager ul').append('<li style="width:' + itemWidth + '%"><a id="layer-' + index + '"data-toggle="tooltip" data-placement="bottom" title="' + value['Date'] + ': ' + value['EVENTO'] + '" data-id="' + value.Code + '" href="#">' + value['Date'] + ': ' + value['EVENTO'] + '</a></li>');
            $('#pager ul li a').tooltip();
        });

        $('.loading').hide();

        $('#pager').slideDown();

        $('#layer-0').trigger('click');
    }


});



$('body').on('click', '.layer-switch li a', function(e) {

    var datos = [];

    e.preventDefault();

    $('.loading').show();
    $('#explorer').hide();
    $('.layer-switch li a').removeClass('active');
    $(this).addClass('active');
    municipalitiesLayer.clearLayers();
    markerLayer.clearLayers();
    var query = $(this).data('id');
    var event = $(this).text();
    $('#event-select-label').text(event);

	$('#projects tr').remove();
	$('#projects').append('<tr><td>Proyecto</td><td>Importe</td></tr>');


    $.ajax({
        type: 'GET',
        url: 'http://catalogo.datos.gob.mx/api/action/datastore_search?resource_id=acbc6f7f-32cd-467d-8ee0-a71b4db7d647&filters={"Code": "' + query + '"}&limit=100000',
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


            map.fitBounds(municipalitiesLayer.getBounds(), {
                maxZoom: 9
            });


        }

    });



    $.ajax({
        type: 'GET',
        url: 'http://201.175.32.249/api/action/datastore_search?resource_id=39e78078-495e-4c0e-a202-4b6668a226b9&filters={"Code": "' + query + '"}&limit=100000',
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
        url: 'http://201.175.32.249/api/action/datastore_search?resource_id=37814da6-8d53-4287-a8c4-86492a636cfb&filters={"Code": "' + query + '"}&limit=100000',
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

});

/*
markerLayer.on('clusterclick', function(e) {
  e.layer.spiderfy();
});
*/

markerLayer.on('click', function(e) {
   
    var targetRecord = '.' + e.layer.options.id;
    var eventId = e.layer.options.eventId;
    $('#explorer').show();
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

var path = window.location.pathname;
$('a#fullscreen').attr('href', path);

</script> 
