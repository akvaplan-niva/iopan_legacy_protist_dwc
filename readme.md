# Marine protist occurrence data 2009-2017

[Darwin Core](https://dwc.tdwg.org) (DwC) data for [npolar.no]'s [marine-db/milestone/1](https://github.com/npolar/marine-db/milestone/1): [Darwin Core Occurrences](https://dwc.tdwg.org/terms/#occurrence), programmatically compiled from original IOPAN protist spreadsheets.

The occurrences are provided as [NDJSON](./ndjson).

The data derives from 1754+ samples from 16 scientific expeditions (see [expeditions-fieldnumbers](./log/expeditions-fieldnumbers.txt) for sample counts per cruise) and amounts to about 37000 counted occurrences of protist cells.

# Overview
```tsv
year	expedition	gear	sampletype	count
2009	Alkekonge2009	Niskin bottle	microplankton	432
2009	Alkekonge2009	Niskin bottle	phytoplankton	1036
2009	MERCLIM-2009	Niskin bottle	microplankton	201
2009	MERCLIM-2009	Niskin bottle	phytoplankton	908
2010	Alkekonge2010	Handnet	handnet	68
2010	Alkekonge2010	Niskin bottle	microplankton	146
2010	Alkekonge2010	Niskin bottle	phytoplankton	747
2010	ICE2010	undefined		152
2011	MOSJ2011	Handnet	handnet	170
2011	MOSJ2011	Niskin bottle	microplankton	550
2011	MOSJ2011	Niskin bottle	phytoplankton	817
2012	ICE2012	Niskin bottle	microplankton	608
2012	ICE2012	Niskin bottle	phytoplankton	2057
2012	MOSJ2012	Handnet	handnet	165
2012	MOSJ2012	Niskin bottle	microplankton	240
2012	MOSJ2012	Niskin bottle		10
2012	MOSJ2012	Niskin bottle	phytoplankton	1223
2012	PolarNight2012	Niskin bottle	microplankton	22
2012	PolarNight2012	Niskin bottle	phytoplankton	18
2013	MOSJ2013	Handnet	handnet	33
2013	MOSJ2013	Niskin bottle	microplankton	174
2013	MOSJ2013	Niskin bottle		3
2013	MOSJ2013	Niskin bottle	phytoplankton	2058
2014	ICE2014	Niskin bottle		2
2014	ICE2014	Niskin bottle	phytoplankton	919
2015	MOSJ2015	handnet	handnet	13
2015	MOSJ2015	handnet	handnet	26
2015	MOSJ2015	Niskin bottle	phytoplankton	233
2015	MOSJ2015	Niskin bottle	phytoplankton	76
2015	N-ICE2015	bottle		69
2015	N-ICE2015	bottle	phytoplankton	16
2015	N-ICE2015	bucket		287
2015	N-ICE2015	bucket	phytoplankton	23
2015	N-ICE2015	core		423
2015	N-ICE2015	core	phytoplankton	429
2015	N-ICE2015	Limnos water sampler	phytoplankton	228
2015	N-ICE2015	Niskin bottle	microplankton	6758
2015	N-ICE2015	Niskin bottle	phytoplankton	3878
2015	N-ICE2015	sediment_trap	phytoplankton	76
2015	N-ICE2015	slurp gun		17
2015	N-ICE2015	Slurp gun		186
2015	N-ICE2015	Slurp gun	phytoplankton	74
2015	N-ICE2015	spoon		35
2015	N-ICE2015	undefined		204
2016	ARCEx2016	Handnet	handnet	155
2016	ARCEx2016	Niskin bottle	phytoplankton	1418
2016	MOSJ2016	Handnet	handnet	303
2016	MOSJ2016	Niskin bottle	microplankton	728
2016	MOSJ2016	Niskin bottle		2
2016	MOSJ2016	Niskin bottle	phytoplankton	3252
2017	GlacierFront2017	handnet	handnet	58
2017	GlacierFront2017	Niskin bottle		1
2017	GlacierFront2017	Niskin bottle	phytoplankton	345
``` 

See also

- [input files](./log/line-counts.txt) with line number counts
- [issues of milestone 1](https://github.com/npolar/marine-db/issues?q=is%3Aissue+milestone%3A%22Original+IOPAN+protist+data+2009-2017+converted+to+Darwin+Core%22)

## Calculations

## Formulas

In the DwC output, `organismQuantity` (protist cells/l) is provided, along with the number of cells counted (`individualCount`) and the volume (`sampleSizeValue`) in litres. It follows that `organismQuantity` may be (re-)calculated by  
`individualCount/sampleSizeValue`.

The counting/sediment chamber volume in `sampleSizeValue` is given by:

```js
initialVolume *
  dilutionFactor *
  (fieldsInCount / maxFields) *
  (takenVolume / bottleVolume);
```

## Validate calculations

A [deno](https://deno.land) script is [provided](./iopan_protist_recalc) to recalculate protist the volume calculations outlined above.

```sh
$ cat ndjson/* | ./iopan_protist_recalc --occurrence | nd-count --select 'perfect'

```

```ndjson
{"perfect":true,"count":30998}
{"perfect":null,"count":5662}
{"perfect":false,"count":243}

When perfect is `null`, the explanation may be that the volume is incalculable (ie. handnet).
Non-perfect records should not be trusted without further investigation.

```

## Taxonomy

[464 uniq spellings of scientific names are are used](./log/taxonomy-uniq-counts.txt)

```sh
$ cat ndjson/* | nd-count --select scientificName | nd-filter 'd.scientificName?.length>0' | gbif-species-api --worms --raw | nd-map 'r0=d.results?.[0]??{}, {key,canonicalName,taxonID,issues,synonym,references}=r0, {scientificName}=d, {scientificName, canonicalName,taxonID,issues,key,synonym,references}' > log/taxonomy-gbif-api-worms.ndjson
```

307 names have a known WoRMS LSID

```sh
che@:~/akvaplan-niva/iopan_legacy_protist_dwc$ cat log/taxonomy-gbif-api-worms.ndjson  | nd-count '{ worms: d?.taxonID?.match(/^urn:lsid:marinespecies\.org/) }'
{"count":156}
{"worms":["urn:lsid:marinespecies.org"],"count":307}
```

## Events

The sampling events for this dataset is published in API v2:
https://v2-api.npolar.no/darwin-core/sample/

At the moment, 51 samples spread over 1183 occurrences have a fieldNumber that is not found in the above API.
List: [error-fieldnumber-not-found.txt](./log/error-fieldnumber-not-found.txt)
