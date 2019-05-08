// $('document').ready(function(){

  var state = {current:""};
  var data = {};
  var table = {};

  $.ajax({
    url:"data/catchments.geojson",
    dataType: "json",
    success: function () {},
    error: function (xhr) {console.log(xhr.statusText);}
  }).done(function(catchments) {

    addThings("catchments",catchments,13)

    $('#thres-slider').on('change click',function() {
      thres(catchments,data.parcels);
    });

    $('.title').click(function() {
      addThings("catchments",catchments,13);
    });
  })

  var addThings = (statename,geojson,zoom) => {

    state.parcels = [];
    data.parcels = [];

    state.current = statename;
    data[statename] = geojson;

    map.eachLayer(function(layer) {
      if (layer != mapBase && layer != mapLabels) {
        map.removeLayer(layer);
      }
    });
    state[statename] = L.geoJSON(geojson, {
      style: style,
      onEachFeature: function(feature,layer) {
        layer.on({
          mouseover: highlightFeature,
          mouseout: resetHighlight,
          click: zoomToFeature
        });
      }
    }).addTo(map);
    map.setMinZoom(zoom);
    map.setMaxBounds(state[statename].getBounds().pad(.25));
    map.fitBounds(state[statename].getBounds().pad(.25));

    thres(data.catchments,data.parcels);

    $('.catchments,.parcels,.individual').addClass('d-none');
    $(`.${statename}`).removeClass('d-none');

    if (statename == "catchments") {
      $('.musa').removeClass('d-none');
      $('.back').addClass('d-none');
    } else {
      $('.musa').addClass('d-none');
      $('.back').removeClass('d-none');
    }


  }

  var thres = (catchments,parcels) => {
    let t = $('#thres-slider').val();

    $(`#parcels, #catchments`).DataTable().destroy();
        $('#catchments tbody, #parcels tbody').html('');

    let total = 0;
    catchments.features.forEach(function(cat) {
      cat = cat.properties;
      count = eval(cat.val_HS).filter(function(val) {return val > t}).length;
      total += count;
      // jquery append table for each catchment
      $('#catchments tbody').append(`
        <tr class="text-nowrap text-right" id="c${cat.CATCH}">
          <td style="padding:0 1rem 0 0.5rem !important;"><h4 class="m-1">${cat.CATCH}</h4></td>
          <td style="padding:0 1rem 0 0.5rem !important;">${count}</td>
          <td style="padding:0 1rem 0 0.5rem !important;">${Math.round(cat.avg_HS*10)/10}</td>
        </tr>
      `);

      $(`#c${cat.CATCH}`).on({
        mouseenter: function() {highlightFeature(match("CATCH",cat.CATCH,state.catchments))},
        mouseleave: function() {resetHighlight(match("CATCH",cat.CATCH,state.catchments))},
        click: function() {zoomToFeature(match("CATCH",cat.CATCH,state.catchments))}
      });

    });

    state.catchments.eachLayer(function (layer) {
      cat = layer.feature.properties;
      layer.bindPopup(`
        <h4 style="text-align:center;border:3px solid ${getColor(cat.avg_HS)}">CATCH ${cat.CATCH}</h4>
        <div class="d-flex flex-col justify-content-between">
          <span class="d-flex">Properties to Inspect &nbsp;</span>
          <span class="d-flex">${eval(cat.val_HS).filter(function(val) {return val > t}).length}</span>
        </div>
        <div class="d-flex flex-col justify-content-between">
          <span class="d-flex">Average Risk Score &nbsp;</span>
          <span class="d-flex">${Math.round(cat.avg_HS*10)/10}</span>
        </div>
      `);
      layer.on('mouseover', function(e){
        layer.openPopup();
      });
    })

    if (state.current == "parcels") {
      state.parcels.eachLayer(function (layer) {
        layer.setStyle(style);
        state.parcels.resetStyle(layer);

        pcl = layer.feature.properties;
        layer.bindPopup(`
          <p>
            <strong>${pcl.address}</strong><br>
            SBL No. ${pcl.SBL}
          </p>
          <p>${pcl.own_name}<br>${pcl.own_town} ${pcl.own_zip}</p>
          <p>
            Past Health/Safety Violations: ${pcl.count_HS}<br>
            Past Total Violations: ${pcl.count_viol}
          </p>
          <p>
            Risk of Health Violation: ${Math.round(pcl.prb_HEALTH)}<br>
            Risk of Safety Violation: ${Math.round(pcl.prb_SAFETY)}
          </p>

        `);
        layer.on('mouseover', function(e){
          layer.openPopup();
        });
      });

      let props = 0;
      parcels.features.forEach(function(pcl) {
        pcl = pcl.properties;
        if (pcl.prb_HS >= t) {props += 1;}
        // jquery append table for each parcel
        $('#parcels tbody').append(`
          <tr class="text-nowrap" id="p${pcl.SBL_class}">
            <td>${pcl.address}</td>
            <td class="text-right">${pcl.prb_HS}</td>
          </tr>
        `);

        $(`#p${pcl.SBL_class}`).on({
          mouseenter: function() {highlightFeature(match("SBL_class",pcl.SBL_class,state.parcels))},
          mouseleave: function() {resetHighlight(match("SBL_class",pcl.SBL_class,state.parcels))},
          click: function() {zoomToFeature(match("SBL_class",pcl.SBL_class,state.parcels))}
        });

      });

      $('.thres-count').text(props);
    }

    $(`#${state.current}`).DataTable({
      "paging"   : false,
      "info"     : false,
      "searching": false
    });

    $('.thres-total').text(total);
    $('.thres-risk').text(t);

    state.threshold = t;

  }

  var legend = L.control({position: 'bottomleft'});

  legend.onAdd = function (map) {

      var div = L.DomUtil.create('div', 'info legend leaflet-bar'),
          grades = [0, 10, 20, 30, 40, 50],
          labels = [];

      div.innerHTML = `
      <h6>Health/Safety <br>Violation Risk</h6>
      `

      // loop through our density intervals and generate a label with a colored square for each interval
      for (var i = 0; i < grades.length; i++) {
          div.innerHTML +=
              '<i style="background:' + getColor(grades[i] + 1) + '"></i> ' +
              grades[i] + (grades[i + 1] ? '&ndash;' + grades[i + 1] + '<br>' : '+');
      }

      return div;
  };

  legend.addTo(map);





  function match(key,value,stateObj) {
    var out = {};
    stateObj.eachLayer(function(layer) {
      layerKey = layer.feature.properties;
      if (layerKey[key] == value) {
        out.target = layer;
      }
    })

    return out;
  }

  function getColor(d) {
    return state.current == "parcels" && d >= state.threshold ? '#ff0000' :
           d > 45 ? '#4d004b' :
           d > 40 ? '#810f7c' :
           d > 35 ? '#88419d' :
           d > 30 ? '#8c6bb1' :
           d > 25 ? '#8c96c6' :
           d > 20 ? '#9ebcda' :
           d > 15 ? '#bfd3e6' :
           d > 10 ? '#e0ecf4' :
           d > 5  ? '#f7fcfd' :
           d > 0  ? '#ffffff' :
                    '#aaaaaa';
  }

  function style(feature) {
    return {
        fillColor: getColor(feature.properties.avg_HS|feature.properties.prb_HS),
        weight: 1,
        opacity: 1,
        color: 'black',
        fillOpacity: 0.7
    };
  }

  function highlightFeature(e) {
    var layer = e.target;

    layer.setStyle({
        color: '#aaa',
        dashArray: '',
        fillOpacity: 0.7
    });

    if (!L.Browser.ie && !L.Browser.opera && !L.Browser.edge) {
        layer.bringToFront();
    }

    let cid = e.target.feature.properties.CATCH;
    $(`#c${cid}`).addClass('hover');

  }

  function resetHighlight(e) {
    state[state.current].resetStyle(e.target);

    let cid = e.target.feature.properties.CATCH;
    $(`#c${cid}`).removeClass('hover');
  }

  function zoomToFeature(e) {
    let cid = e.target.feature.properties.CATCH;

    if (state.current == "catchments") {
      $.ajax({
        url:`data/catch${cid}.geojson`,
        dataType: "json",
        success: function () {},
        error: function (xhr) {console.log(xhr.statusText);}
      }).done(function(parcels) {

        parcels.features.forEach(function(pcl) {
          pcl = pcl.properties;
          pcl.SBL_class = pcl.SBL.replace(/[^\w\s]/gi, '');
          pcl.prb_HS = (pcl.prb_HS === "NA") ? -1 : Math.round(pcl.prb_HS);
        });

        addThings('parcels',parcels,15);
        $('.parcels .cid').text(`${cid}`);
      });
    }

    if (state.current == "parcels") {
      map.fitBounds(e.target.getBounds());
    }

  }

// });
