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

This module also assumes that OpenSIPS is started in the UTC timezone.
(Using `TZ=UTC` in the startup script should accomplish this.)
