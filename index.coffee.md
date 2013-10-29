A toolbox for OpenSIPS
======================

Install
-------

```
npm install opensips
```


`db_http` module helpers
------------------------

This module assumes your opensips.cfg contains the following:

```
modparams("db_http","field_delimiter","\t")
modparams("db_http","quote_delimiter","\"")
modparams("db_http","row_delimiter","\n")
```

    field_delimiter = '\t'
    row_delimiter = '\n'
    quote_delimiter = '"'
    null_field = '\\0'

    @unquote_value = unquote_value = (t,x) ->
      if not x?
        return x

      if t is 'int'
        return parseInt(x)
      if t is 'double'
        return parseFloat(x)
      # Not sure what the issue is, but we're getting garbage at the end of dates.
      if t is 'date'
        d = new Date(x)
        # Format expected by db_str2time() in db/db_ut.c
        # Note: This requires opensips to be started in UTC, assuming
        #       toISOString() outputs using UTC (which it does in Node.js 0.4.11).
        #       Our script ccnq3-opensips.postinst makes sure this is the case.

This module also assumes that OpenSIPS is started in the UTC timezone.
(Using `TZ=UTC` in the startup script should accomplish this.)

        return d.toISOString().replace 'T', ' '

      # string, blob, ...
      return x.toString()

    @unquote_params = (k,v,types)->
      doc = {}
      names = k.split ','
      values = v.split ','

      doc[names[i]] = unquote_value(types[names[i]],values[i]) for i in [0..names.length]

      return doc

    @line = (a) ->
      a.join(field_delimiter) + row_delimiter

    @header = (columns,types) ->
      columns.map (c) -> types[c] ? 'string'

    @data = (columns,doc) ->
      columns.map (c) -> doc[c] ? null_field

    @location_types =
      expires: 'date'
      q: 'double'
      cseq: 'int'
      flags: 'int'
      methods: 'int'
      last_modified: 'date'
