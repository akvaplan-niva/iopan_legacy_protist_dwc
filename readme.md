# IOPAN legacy data protist data 2009-2017

Data for [npolar/marine-db/milestone/1](https://github.com/npolar/marine-db/milestone/1).

See also

- [input files](https://github.com/npolar/marine-db/issues/56)
- [issues of milestone 1](https://github.com/npolar/marine-db/issues?q=is%3Aissue+milestone%3A%22Original+IOPAN+protist+data+2009-2017+converted+to+Darwin+Core%22)

## Calculations

## Formula

The `organismQuantity` in the output data (protist cells/l) may be calculated by `individualCount/sampleSizeValue`. The volume in litres (`sampleSizeValue`) is given by:

```js
volume =
  initialVolume *
  dilutionFactor *
  (fieldsInCount / maxFields) *
  (takenVolume / bottleVolume);
```

## Validate calculations

A [https://deno](deno.land) script is [provided](./iopan_protist_recalc) to recalculate and validate Darwin Core occurrence NDJSON from stdin

### perfect

```sh
$ cat ndjson/* | ./iopan_protist_recalc  | nd-count --select perfect
```

```ndjson
{"perfect":true,"count":30423}
{"perfect":null,"count":5662}
{"perfect":false,"count":818}
```

### ratios

```
$ cat ndjson/* | ./iopan_protist_recalc | nd-count --select volumeRatio
```

```ndjson

{"volumeRatio":1,"count":31161}
{"volumeRatio":null,"count":5662}
{"volumeRatio":0.92,"count":56}
{"volumeRatio":0.1,"count":20}
{"volumeRatio":0.28,"count":4}

```

```sh
cat ndjson/* | ./iopan_protist_recalc | nd-count --select quantityRatio | nd-group 'parseInt(values(d)[0])' | nd-map '[d[0], d[1].map(({count})=>count)]'
```

```ndjson
[1,[30499,2,2,2,1]]
[null,[5666]]
[2,[18,1,1]]
[0,[597,1,13,2,10,6,1,2,2,1,2,1,1,1,2,1,1,1,2,1,2,1,1,1,1,1,1,1,18]]
[8,[2]]
[4,[2,2,1]]
[3,[7,1]]
[79,[1]]
[9,[2]]
[6,[3,1]]
[18,[1]]
[5,[1,1,1]]
[51,[1]]
[7,[1,1]]
[36,[1]]
[20,[1]]
[13000,[1]]
[25000,[1]]
[6300,[1]]
[57000,[1]]
[11,[1]]
```

## Taxonomy

## Events
