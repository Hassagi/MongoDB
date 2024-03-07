## Analiza danych dotyczących zanieczyszczenia powietrza

### 1. Import danych

```bash
mongoimport --jsonArray -d aq -c data --type json --file /home/hasanga/Downloads/openaq.json
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

### 4. Usuń rekordy z ujemnymi wartościami zanieczyszczenia

```javascript
var errors = { 'measurements_value': { $lt: 0 } };
db.data.deleteMany(errors);
```

### 5. Unikalne typy zanieczyszczeń i analiza ilości i średniej wartości

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
// MapReduce
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

Wyniki:

```
[ { _id: 'Southern', count: 3288 }, { _id: 'Northern', count: 49420 } ]
[ { _id: 'Western', count: 13647 }, { _id: 'Eastern', count: 39061 } ]
```
