# MongoDB

1. Zaimportuj dane z adresu URL do kolekcji data bazy aq.

mongoimport --jsonArray -d aq -c data --type json --file path_to_json_file
2024-02-29T20:00:10.274+0100	connected to: mongodb://localhost/
2024-02-29T20:00:13.260+0100	53407 document(s) imported successfully. 0 document(s) failed to import.

Wyjaśnienie: Komenda mongoimport służy do importowania danych w formacie JSON do bazy danych MongoDB. W tym przypadku, dane z pliku openaq.json zostały zaimportowane do kolekcji data w bazie aq.

2. Wypisz unikalne kraje, z których pochodzą dane o zanieczyszczeniu powietrza

db.data.distinct('country_name_en')
[
  null,
  'Afghanistan',
  'Algeria',
  'Andorra',
  'Antigua and Barbuda',
  'Argentina',
  'Armenia',
  'Australia',
  'Austria',
  'Azerbaijan',
  'Bahrain',
  'Bangladesh',
  'Belgium',
  'Belize',
  'Bermuda',
  'Bosnia and Herzegovina',
  'Brazil',
  'Bulgaria',
  'Canada',
  'Central African Republic',
  'Chad',
  'Chile',
  'China',
  'Colombia',
  'Congo, Democratic Republic of the',
  'Costa Rica',
  'Croatia',
  'Cyprus',
  'Czech Republic',
  "Côte d'Ivoire",
  'Denmark',
  'Ecuador',
  'Egypt',
  'Estonia',
  'Ethiopia',
  'Finland',
  'France',
  'Gabon',
  'Germany',
  'Ghana',
  'Gibraltar',
  'Greece',
  'Guatemala',
  'Guinea',
  'Hong Kong, China',
  'Hungary',
  'Iceland',
  'India',
  'Indonesia',
  'Iraq',
  'Ireland',
  'Israel',
  'Italy',
  'Japan',
  'Jordan',
  'Kazakhstan',
  'Kenya',
  'Korea, Republic of',
  'Kuwait',
  'Kyrgyzstan',
  "Lao People's Dem. Rep.",
  'Latvia',
  'Lithuania',
  'Luxembourg',
  'Macedonia, The former Yugoslav Rep. of',
  'Madagascar',
  'Malaysia',
  'Mali',
  'Malta',
  'Mexico',
  'Moldova, Republic of',
  'Mongolia',
  'Montenegro',
  'Morocco',
  'Mozambique',
  'Myanmar',
  'Nepal',
  'Netherlands',
  'New Zealand',
  'Nicaragua',
  'Nigeria',
  'Norway',
  'Pakistan',
  'Peru',
  'Philippines',
  'Poland',
  'Portugal',
  'Qatar',
  'Romania',
  'Russian Federation',
  'Rwanda',
  'Saudi Arabia',
  'Serbia',
  'Serbia and Montenegro',
  'Singapore',
  'Slovakia',
  'Slovenia',
  'South Africa',
  'Spain',
  'Sri Lanka',
  ... 16 more items
]

Wyjaśnienie: Wykorzystując metodę distinct(), pobierane są unikalne wartości pola country_name_en z kolekcji data.

3. Zlicz liczbę unikalnych krajów, miast i lokalizacji w danych.

db.data.distinct('country_name_en').length
116

db.data.distinct('city').length
2341

db.data.distinct('location').length
10290

Wyjaśnienie: Wykorzystując metodę distinct(), zliczane są unikalne wartości poszczególnych pól w kolekcji data.
Wyczyszczenie danych z niepoprawnych rekordów (ujemne wartości zanieczyszczenia):

4. Usuń rekordy z ujemnymi wartościami zanieczyszczenia.

var errors = { 'measurements_value': { $lt: 0 } };
db.data.deleteMany(errors);
{ acknowledged: true, deletedCount: 699 }


Wyjaśnienie: Przy użyciu deleteMany() usuwane są rekordy z kolekcji data, które zawierają wartości ujemne w polu measurements_value.

5. Wypisz unikalne typy zanieczyszczeń i wykonaj analizę ilości i średniej wartości dla każdego typu zanieczyszczenia.

db.data.distinct('measurements_parameter')
[
  'BC',          'CO',
  'NO',          'NO2',
  'NOX',         'O3',
  'PM1',         'PM10',
  'PM2.5',       'SO2',
  'TEMPERATURE'
]

db.data.aggregate([
  { $group: { _id: '$measurements_parameter', count: { $count: {} }, avgValue: { $avg: '$measurements_value' } } }
])
[
  { _id: 'NO', count: 3662, avgValue: 4.7210016502278185 },
  { _id: 'SO2', count: 6757, avgValue: 6.129378192742367 },
  { _id: 'NOX', count: 2007, avgValue: 1.4551807251895532 },
  { _id: 'CO', count: 5008, avgValue: 6599.165299722113 },
  { _id: 'PM1', count: 9, avgValue: 9.676592222222222 },
  { _id: 'BC', count: 98, avgValue: 0.5969273469387755 },
  { _id: 'TEMPERATURE', count: 86, avgValue: 18.00790411834149 },
  { _id: 'O3', count: 8293, avgValue: 48.3562594437179 },
  { _id: 'PM2.5', count: 9326, avgValue: 17.95696136334306 },
  { _id: 'PM10', count: 7785, avgValue: 36.852504569305495 },
  { _id: 'NO2', count: 9677, avgValue: 14.777163816961982 }
]

Wyjaśnienie: Wykorzystując metodę distinct(), pobierane są unikalne wartości pola measurements_parameter. Następnie, korzystając z agregacji, grupowane są dane według typu zanieczyszczenia, a dla każdego typu obliczana jest liczba próbek oraz średnia wartość zanieczyszczenia.

6. Podaj najwyższe zarejestrowane stężenia pyłków PM10 i PM2.5 dla każdego kraju.

db.data.aggregate([
  { $match: { 'measurements_parameter': 'PM10' } },
  { $group: { _id: '$country_name_en', maxValue: { $max: '$measurements_value' } } }
])
[
  { _id: 'Luxembourg', maxValue: 21 },
  { _id: 'Chile', maxValue: 1000 },
  { _id: 'Macedonia, The former Yugoslav Rep. of', maxValue: 66.22 },
  { _id: 'Mexico', maxValue: 987.76 },
  { _id: 'Slovenia', maxValue: 37 },
  { _id: 'Latvia', maxValue: 20 },
  { _id: 'Finland', maxValue: 11.8548762 },
  { _id: 'Bulgaria', maxValue: 70.59 },
  { _id: 'Croatia', maxValue: 28.1 },
  { _id: 'Ireland', maxValue: 20.88 },
  { _id: 'South Africa', maxValue: 999.19 },
  { _id: 'Austria', maxValue: 83.33000183 },
  { _id: 'Bosnia and Herzegovina', maxValue: 101 },
  { _id: 'Denmark', maxValue: 31.7 },
  { _id: 'Hong Kong, China', maxValue: 69.4 },
  { _id: 'Cyprus', maxValue: 39.47 },
  { _id: 'Iceland', maxValue: 988 },
  { _id: 'New Zealand', maxValue: 21.018713 },
  { _id: 'India', maxValue: 1985 },
  { _id: 'Poland', maxValue: 59.4675 }
]

db.data.aggregate([
  { $match: { 'measurements_parameter': 'PM2.5' } },
  { $group: { _id: '$country_name_en', maxValue: { $max: '$measurements_value' } } }
])
[
  { _id: 'Luxembourg', maxValue: 5 },
  { _id: 'Chile', maxValue: 100 },
  { _id: 'Moldova, Republic of', maxValue: 18 },
  { _id: 'Macedonia, The former Yugoslav Rep. of', maxValue: 52.31 },
  { _id: 'Madagascar', maxValue: 17 },
  { _id: 'Sri Lanka', maxValue: 39 },
  { _id: 'Mexico', maxValue: 985 },
  { _id: 'Latvia', maxValue: 17 },
  { _id: 'Finland', maxValue: 6.9647159 },
  { _id: 'Bulgaria', maxValue: 29.51 },
  { _id: 'Azerbaijan', maxValue: 12 },
  { _id: 'Croatia', maxValue: 16.8 },
  { _id: 'Ireland', maxValue: 14.22 },
  { _id: 'South Africa', maxValue: 997.71 },
  { _id: 'Austria', maxValue: 32.75 },
  { _id: 'Mozambique', maxValue: 4 },
  { _id: 'Bosnia and Herzegovina', maxValue: 87 },
  { _id: 'Denmark', maxValue: 1.6279761989911397 },
  { _id: 'Hong Kong, China', maxValue: 46.1 },
  { _id: 'Estonia', maxValue: 8 }
]

Wyjaśnienie: Korzystając z agregacji, najpierw filtrowane są dane dla pyłków PM10 i PM2.5, a następnie grupowane są według krajów, aby znaleźć maksymalne wartości stężeń dla każdego typu pyłków.

7. Z wykorzystaniem MapReduce zlicz i podaj średnią wartość dla próbek, z podziałem na typy zanieczyszczeń.

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
[
  {
    _id: { measurements_parameter: 'CO' },
    value: { measurements_value: 33048619.821008347, avg: 6599.165299722114 }
  },
  {
    _id: { measurements_parameter: 'SO2' },
    value: { measurements_value: 41416.2084483599, avg: 6.129378192742327 }
  },
  {
    _id: { measurements_parameter: 'PM1' },
    value: { measurements_value: 87.08933, avg: 9.676592222222222 }
  },
  {
    _id: { measurements_parameter: 'PM10' },
    value: { measurements_value: 286896.7480720442, avg: 36.852504569305616 }
  },
  {
    _id: { measurements_parameter: 'O3' },
    value: { measurements_value: 401018.45956675266, avg: 48.35625944371792 }
  },
  {
    _id: { measurements_parameter: 'NO' },
    value: { measurements_value: 17288.30804313448, avg: 4.721001650227875 }
  },
  {
    _id: { measurements_parameter: 'BC' },
    value: { measurements_value: 58.49887999999999, avg: 0.5969273469387755 }
  },
  {
    _id: { measurements_parameter: 'NOX' },
    value: { measurements_value: 2920.5477154554346, avg: 1.4551807251895539 }
  },
  {
    _id: { measurements_parameter: 'NO2' },
    value: { measurements_value: 142998.61425674055, avg: 14.777163816961925 }
  },
  {
    _id: { measurements_parameter: 'PM2.5' },
    value: { measurements_value: 167466.62167453754, avg: 17.956961363343076 }
  },
  {
    _id: { measurements_parameter: 'TEMPERATURE' },
    value: { measurements_value: 1548.6797541773678, avg: 18.007904118341486 }
  }
]

Wyjaśnienie: Wykorzystując MapReduce, grupowane są dane według typu zanieczyszczenia, a następnie obliczana jest suma wartości oraz liczba próbek dla każdego typu. Na końcu obliczana jest średnia wartość zanieczyszczenia dla każdego typu.

8. Z wykorzystaniem metody aggregate zlicz i podaj średnią wartość dla próbek, z podziałem na typy zanieczyszczeń.

db.data.aggregate([
  { $group: { _id: '$measurements_parameter', avg: { $avg: '$measurements_value' } } }
])
[
  { _id: 'PM10', avg: 36.852504569305495 },
  { _id: 'BC', avg: 0.5969273469387755 },
  { _id: 'PM2.5', avg: 17.95696136334306 },
  { _id: 'NO', avg: 4.7210016502278185 },
  { _id: 'NO2', avg: 14.777163816961982 },
  { _id: 'NOX', avg: 1.4551807251895532 },
  { _id: 'CO', avg: 6599.165299722113 },
  { _id: 'PM1', avg: 9.676592222222222 },
  { _id: 'SO2', avg: 6.129378192742367 },
  { _id: 'TEMPERATURE', avg: 18.00790411834149 },
  { _id: 'O3', avg: 48.3562594437179 }
]

Wyjaśnienie: Wykorzystując metodę aggregate(), grupowane są dane według typu zanieczyszczenia, a następnie obliczana jest średnia wartość zanieczyszczenia dla każdego typu.

9. Podaj liczbę punktów pomiaru powietrza na każdej półkuli.

Zapytanie 1: Podział na Północ i Południe

db.data.aggregate([
  {
    $project: {
      _id: 0,
      latitude: "$coordinates.lat",
      hemisphere: {
        $cond: [
          { $gte: ["$coordinates.lat", 0] },
          "Northern",
          "Southern"
        ]
      }
    }
  },
  {
    $group: {
      _id: "$hemisphere",
      count: { $sum: 1 }
    }
  }
])

[ { _id: 'Southern', count: 3288 }, { _id: 'Northern', count: 49420 } ]

To zapytanie podzieli dane na północną i południową półkulę na podstawie wartości szerokości geograficznej (latitude). Następnie grupuje dane według tych kierunków geograficznych i oblicza liczbę punktów pomiarowych w każdej grupie.

Zapytanie 2: Podział na Wschód i Zachód

db.data.aggregate([
  {
    $project: {
      _id: 0,
      longitude: "$coordinates.lon",
      hemisphere: {
        $cond: [
          { $gte: ["$coordinates.lon", 0] },
          "Eastern",
          "Western"
        ]
      }
    }
  },
  {
    $group: {
      _id: "$hemisphere",
      count: { $sum: 1 }
    }
  }
])

[ { _id: 'Western', count: 13647 }, { _id: 'Eastern', count: 39061 } ]

To zapytanie podzieli dane na wschodnią i zachodnią półkulę na podstawie wartości długości geograficznej (longitude). Następnie grupuje dane według tych kierunków geograficznych i oblicza liczbę punktów pomiarowych w każdej grupie.

