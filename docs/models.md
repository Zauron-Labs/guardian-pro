# AI Models

Guardian Pro leverages purpose-built AI models to identify quality-relevant findings across a range of imaging modalities and body parts. The table below lists all currently supported models along with their validated performance characteristics.

**Minimum Positive Predictive Value (PPV)** represents the lowest acceptable precision threshold validated on clinical data. Models at or above this threshold are deployed for case selection.

<style>
.model-table-wrap {
  width: 100%;
  overflow-x: auto;
  margin: 1.5em 0;
}
.model-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.95em;
}
.model-table th,
.model-table td {
  border: 1px solid #ddd;
  padding: 10px 14px;
  text-align: left;
  white-space: nowrap;
}
.model-table th {
  background: #f5f5f5;
  cursor: pointer;
  user-select: none;
  position: relative;
  font-weight: 600;
}
.model-table th .sort-icon {
  margin-left: 6px;
  opacity: 0.3;
  font-size: 0.75em;
}
.model-table th.sort-asc .sort-icon,
.model-table th.sort-desc .sort-icon {
  opacity: 1;
}
.model-table td:first-child {
  white-space: normal;
}
.model-table tbody tr:hover {
  background: #f9f9f9;
}
.model-filter-row input,
.model-filter-row select {
  width: 100%;
  padding: 5px 8px;
  border: 1px solid #ccc;
  border-radius: 3px;
  font-size: 0.85em;
  box-sizing: border-box;
}
.model-filter-row td {
  background: #fafafa;
  padding: 6px 10px;
}
.badge {
  display: inline-block;
  padding: 2px 10px;
  border-radius: 12px;
  font-size: 0.85em;
  font-weight: 500;
}
.badge-open {
  background: #e6f4ea;
  color: #1e7e34;
}
.badge-closed {
  background: #fce8e6;
  color: #c5221f;
}
</style>

<div class="model-table-wrap">
<table class="model-table" id="modelTable">
  <thead>
    <tr>
      <th data-col="0" data-type="string" class="sort-asc">Abnormality Type <span class="sort-icon">▲</span></th>
      <th data-col="1" data-type="string">Modality <span class="sort-icon">▲</span></th>
      <th data-col="2" data-type="string">Body Part <span class="sort-icon">▲</span></th>
      <th data-col="3" data-type="number">Min PPV <span class="sort-icon">▲</span></th>
      <th data-col="4" data-type="string">Source <span class="sort-icon">▲</span></th>
    </tr>
    <tr class="model-filter-row">
      <td><input type="text" id="filter-0" placeholder="Search…" /></td>
      <td>
        <select id="filter-1">
          <option value="">All</option>
          <option value="CT">CT</option>
          <option value="MG">MG</option>
          <option value="MR">MR</option>
          <option value="XR">XR</option>
        </select>
      </td>
      <td>
        <select id="filter-2">
          <option value="">All</option>
          <option value="Breast">Breast</option>
          <option value="Chest">Chest</option>
          <option value="Head">Head</option>
          <option value="Lumbar Spine">Lumbar Spine</option>
        </select>
      </td>
      <td><input type="text" id="filter-3" placeholder="Search…" /></td>
      <td>
        <select id="filter-4">
          <option value="">All</option>
          <option value="Open">Open</option>
          <option value="Closed">Closed</option>
        </select>
      </td>
    </tr>
  </thead>
  <tbody id="modelBody">
  </tbody>
</table>
</div>

<script>
(function () {
  var DATA = [
    ["Breast Cancer",                            "MG", "Breast",       0.25, "Open"],
    ["Cardiomegaly, Pleural Effusion, Pneumonia", "XR", "Chest",       0.75, "Open"],
    ["Coronary Artery Calcium",                   "CT", "Chest",       0.75, "Open"],
    ["Disc degeneration",                         "MR", "Lumbar Spine", 0.75, "Closed"],
    ["Intracranial hemorrhage",                   "CT", "Head",        0.25, "Open"],
    ["Ischemia",                                  "MR", "Head",        0.25, "Closed"],
    ["Pneumothorax",                              "XR", "Chest",       0.25, "Open"],
    ["Pulmonary embolism",                        "CT", "Chest",       0.25, "Open"],
    ["Vertebral fracture",                        "XR", "Chest",       0.25, "Open"]
  ];

  var table   = document.getElementById("modelTable");
  var tbody   = document.getElementById("modelBody");
  var headers = table.querySelectorAll("thead tr:first-child th");
  var sortCol = 0;
  var sortAsc = true;

  function badge(val) {
    var cls = val === "Open" ? "badge-open" : "badge-closed";
    return '<span class="badge ' + cls + '">' + val + "</span>";
  }

  function render(rows) {
    var html = "";
    for (var i = 0; i < rows.length; i++) {
      var r = rows[i];
      html += "<tr>" +
        "<td>" + r[0] + "</td>" +
        "<td>" + r[1] + "</td>" +
        "<td>" + r[2] + "</td>" +
        "<td>" + r[3] + "</td>" +
        "<td>" + badge(r[4]) + "</td>" +
        "</tr>";
    }
    tbody.innerHTML = html || '<tr><td colspan="5" style="text-align:center;padding:20px;">No matching models.</td></tr>';
  }

  function getFiltered() {
    return DATA.filter(function (row) {
      for (var c = 0; c < 5; c++) {
        var el = document.getElementById("filter-" + c);
        var val = el.value.trim().toLowerCase();
        if (!val) continue;
        var cell = String(row[c]).toLowerCase();
        if (el.tagName === "SELECT") {
          if (cell !== val) return false;
        } else {
          if (cell.indexOf(val) === -1) return false;
        }
      }
      return true;
    });
  }

  function getSorted(rows) {
    var type = headers[sortCol].getAttribute("data-type");
    var copy = rows.slice();
    copy.sort(function (a, b) {
      var va = a[sortCol], vb = b[sortCol];
      if (type === "number") {
        return sortAsc ? va - vb : vb - va;
      }
      va = String(va).toLowerCase();
      vb = String(vb).toLowerCase();
      if (va < vb) return sortAsc ? -1 : 1;
      if (va > vb) return sortAsc ? 1 : -1;
      return 0;
    });
    return copy;
  }

  function updateSortUI() {
    for (var i = 0; i < headers.length; i++) {
      headers[i].classList.remove("sort-asc", "sort-desc");
      headers[i].querySelector(".sort-icon").textContent = "▲";
    }
    headers[sortCol].classList.add(sortAsc ? "sort-asc" : "sort-desc");
    headers[sortCol].querySelector(".sort-icon").textContent = sortAsc ? "▲" : "▼";
  }

  function refresh() {
    render(getSorted(getFiltered()));
    updateSortUI();
  }

  for (var i = 0; i < headers.length; i++) {
    (function (idx) {
      headers[idx].addEventListener("click", function () {
        if (sortCol === idx) { sortAsc = !sortAsc; }
        else { sortCol = idx; sortAsc = true; }
        refresh();
      });
    })(i);
  }

  for (var j = 0; j < 5; j++) {
    document.getElementById("filter-" + j).addEventListener("input", refresh);
    document.getElementById("filter-" + j).addEventListener("change", refresh);
  }

  refresh();
})();
</script>
