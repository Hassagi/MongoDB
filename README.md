## Analiza danych dotyczących zanieczyszczenia powietrza

### 1. Import danych

```bash
mongoimport --jsonArray -d aq -c data --type json --file path_to_json_file
```

### 2. Unikalne kraje

```javascript
db.data.distinct('country_name_en')
```

### 3. Liczba unikalnych krajów, miast i lokalizacji

```javascript
db.data.distinct('country_name_en').length
db.data.distinct('city').length
db.data.distinct('location').length
```

### 4. Usuwanie rekordów z ujemnymi wartościami zanieczyszczenia

```javascript
var errors = { 'measurements_value': { $lt: 0 } };
db.data.deleteMany(errors);
```

### 5. Unikalne typy zanieczyszczeń, analiza ilości i średniej wartości

```javascript
db.data.distinct('measurements_parameter')
db.data.aggregate([
  { $group: { _id: '$measurements_parameter', count: { $count: {} }, avgValue: { $avg: '$measurements_value' } } }
])
```

### 6. Najwyższe zarejestrowane stężenia pyłków PM10 i PM2.5 dla każdego kraju

```javascript
db.data.aggregate([
  { $match: { 'measurements_parameter': 'PM10' } },
  { $group: { _id: '$country_name_en', maxValue: { $max: '$measurements_value' } } }
])

db.data.aggregate([
  { $match: { 'measurements_parameter': 'PM2.5' } },
  { $group: { _id: '$country_name_en', maxValue: { $max: '$measurements_value' } } }
])
```

### 7. Zliczanie i średnia wartość dla próbek z podziałem na typy zanieczyszczeń (MapReduce)

```javascript
map = function() {
  emit(
    { measurements_parameter: this.measurements_parameter },
    { measurements_value: this.measurements_value, count: 1 }
  );
}
reduce = function(key, values) {
  var total = 0;
  var count = 0;
  for (var i = 0; i < values.length; i++) {
    total += values[i].measurements_value;
    count += values[i].count;
  }
  return { measurements_value: total, count: count };
}
finalize = function(key, reducedValue) {
  reducedValue.avg = parseInt(reducedValue.measurements_value / reducedValue.count);
  delete reducedValue.count;
  return reducedValue;
}
db.runCommand({
  mapReduce: 'data',
  map: map,
  reduce: reduce,
  out: 'data.report',
  finalize: finalize
})

db.data.report.find()
```

### 8. Zliczanie i średnia wartość dla próbek z podziałem na typy zanieczyszczeń (Agregacja)

```javascript
db.data.aggregate([
  { $group: { _id: '$measurements_parameter', avg: { $avg: '$measurements_value' } } }
])
```

### 9. Liczba punktów pomiaru powietrza na każdej półkuli

#### Podział na Północ i Południe

```javascript
db.data.aggregate([
  { $project: { _id: 0, latitude: "$coordinates.lat", hemisphere: { $cond: [ { $gte: ["$coordinates.lat", 0] }, "Northern", "Southern" ] } } },
  { $group: { _id: "$hemisphere", count: { $sum: 1 } } }
])
```

#### Podział na Wschód i Zachód

```javascript
db.data.aggregate([
  { $project: { _id: 0, longitude: "$coordinates.lon", hemisphere: { $cond: [ { $gte: ["$coordinates.lon", 0] }, "Eastern", "Western" ] } } },
  { $group: { _id: "$hemisphere", count: { $sum: 1 } } }
])
```
