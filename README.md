# Portable-Geo-API
This Project aims to provide an Rest API for the calculation of routes (also isochrones + more to follow) on top of OSM data.

<b>Notice: This project is experimental and under active development right now, meaning refactoring, restructuring, etc will occur. Query format will most certainly change as well. Also the scope is not clear yet so it will probably end up being a more generic geo api which also provides services like isochrone calculation, etc</b>

<h2>General Info</h2>

The API is built in Flask and utilizes PostgreSQL with the extensions PostGIS and pgRouting to perform the spatial calculations. The routing graph is created via osm2po right now as it is able to process a whole planet.pbf dump file (https://planet.osm.org/pbf/planet-latest.osm.pbf) in one run.

For portablility and ease of use the whole framework can be deployed using docker.

<h2>Installation Notes</h2>

<h3>Snapshot DB</h3>

```bash
pg_dump --format=c --create --no-owner --no-privileges --verbose --file=/halde/routing_graph_alpha.dump --dbname=osm_routing
```

<h3>Graph building</h3>

Walkers guide schema: https://raw.githubusercontent.com/scheibler/WalkersGuide-Server/master/misc/osm2po_5.0.18.config
```bash
java -Xmx30g -jar osm2po-core-5.2.43-signed.jar prefix=world tileSize=45x45,1 ../planet-latest.osm.pbf
```

<h3>Create DB</h3>

```bash
DB="osm_routing"; dropdb $DB; createdb $DB; psql -d $DB -c "BEGIN; CREATE EXTENSION postgis; CREATE EXTENSION pgRouting; END;"; logFile="/tmp/routing_sql.log";
```
  
<h3>Load data into DB</h3>

```bash
DB="osm_routing"; logFile="/tmp/routing_sql.log"; rm $logFile; touch $logFile; IFS=$'\n'; for filename in $(ls -1 /halde/split_osm/osm2po/world/*.sql); do (psql -d osm_routing --quiet --file $filename 2>&1 | sed -e "s#^#$(date) ${filename} ($$): #g" | tee -a $logFile) & done;
```

<h3>DB ops</h3>

```sql
create extension postgis;
create extension pgRouting;
create index on world_2po_4pgr using gist (geom_vertex);
create index on world_2po_4pgr using gist (geom_way);
analyze world_2po_4pgr;
cluster world_2po_vertex using world_2po_vertex_geom_vertex_idx;
```

<h3>Documentation and Sandbox</h3>

After deployment visit: http(s)://yourhost:5000
