Infer SQL queries from plain-text questions and table headers.

Requirements:
 * install `docker`
 * install `curl`
 * Make sure docker allows at least 3GB of RAM (see `Docker`>`Preferences`>`Advanced`
   or equivalent)

## sqlova

This wraps up a published pretrained model for Sqlova (https://github.com/naver/sqlova/).

Fetch and start sqlova running as an api server on port 5050:

```
docker run --name sqlova -d -p 5050:5050 paulfitz/sqlova
```

Be patient, the image is about 4.2GB.  Once it is running, it'll take a few seconds
to load models and then you can start asking questions about CSV tables.  For example:

```
curl -F "csv=@bridges.csv" -F "q=how long is throgs neck" localhost:5050
# {"answer":[1800],"params":["throgs neck"],"sql":"SELECT (length) FROM bridges WHERE bridge = ?"}
```

This is using the sample `bridges.csv` included in this repo.

| bridge | designer | length |
|---|---|---|
| Brooklyn | J. A. Roebling | 1595 |
| Manhattan | G. Lindenthal | 1470 |
| Williamsburg | L. L. Buck | 1600 |
| Queensborough | Palmer & Hornbostel | 1182 |
| Triborough | O. H. Ammann | 1380,383 |
| Bronx Whitestone | O. H. Ammann | 2300 |
| Throgs Neck | O. H. Ammann | 1800 |
| George Washington | O. H. Ammann | 3500 |

Here are some examples of the answers and sql inferred for plain-text questions about
this table:

| question | answer | sql |
|---|---|---|
| how long is throgs neck | 1800 | `SELECT (length) FROM bridges WHERE bridge = ? ['throgs neck']` |
| who designed the george washington | O. H. Ammann | `SELECT (designer) FROM bridges WHERE bridge = ? ['george washington']` |
| how many bridges are there | 8 | `SELECT count(bridge) FROM bridges` |
| how many bridges are designed by O. H. Ammann | 4 | `SELECT count(bridge) FROM bridges WHERE designer = ? ['O. H. Ammann']` |
| which bridge are longer than 2000 | Bronx Whitestone, George Washington | `SELECT (bridge) FROM bridges WHERE length > ? ['2000']` |
| how many bridges are longer than 2000 | 2 | `SELECT count(bridge) FROM bridges WHERE length > ? ['2000']` |
| what is the shortest length | 1182 | `SELECT min(length) FROM bridges` |

Some questions about [iris.csv](https://en.wikipedia.org/wiki/Iris_flower_data_set):

| question | answer | sql |
|---|---|---|
what is the average petal width for virginica | 2.026 | SELECT avg(Petal.Width) FROM iris WHERE Species = ? ['virginica'] |
what is the longest sepal for versicolor | 7.0 | SELECT max(Sepal.Length) FROM iris WHERE Species = ? ['versicolor'] |
how many setosa rows are there | 50 | SELECT count(col0) FROM iris WHERE Species = ? ['setosa'] |

There are plenty of types of questions this model cannot answer (and that aren't covered
in the dataset it is trained on, or in the sql it is permitted to generate).  I hope to
track research in the area and substitute in models as they become available.  Things are
moving fast!

