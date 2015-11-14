Tests for `opensips` module
===========================

    opensips = require '../'

    should = require 'should'
    vows = require 'vows'

    process.on 'uncaughtException', (err) ->
      console.log "Caught exception: #{err}"

    suite = vows.describe 'The opensips module'
    suite.addBatch
      '`unquote_value`':
        'parses an integer':
          topic: -> value: opensips.unquote_value 'int', '42'
          'and returns it': (the) ->
            the.value.should.equal 42

        'parses an integer as double':
          topic: -> value: opensips.unquote_value 'double', '42'
          'and returns it': (the) ->
            the.value.should.equal 42

        'parses a float as double':
          topic: -> value: opensips.unquote_value 'double', '3.14'
          'and returns it': (the) ->
            the.value.should.equal 3.14

        'parses dates':
          topic: -> value: opensips.unquote_value 'date','2013-11-20T17:00:00'
          'and returns it': (the) ->
            the.value.should.equal '2013-11-20 17:00:00.000Z'

        'parses strings':
          topic: -> value: opensips.unquote_value 'string', 'foo'
          'and returns it': (the) ->
            the.value.should.equal 'foo'

        'parses numbers as strings':
          topic: -> value: opensips.unquote_value 'string', 42
          'and returns it': (the) ->
            the.value.should.equal '42'

      'contains `unquote_params`':
        'which is a function': (topic) ->
          opensips.unquote_params.should.type 'function'

      'contains `line`':
        'which is a function': (topic) ->
          opensips.line.should.type 'function'
      'contains `header`':
        'which is a function': (topic) ->
          opensips.header.should.type 'function'
      'contains `location_types`':
        topic: opensips.location_types
        'which is a function': (topic) ->
          # assert.equal typeof topic, 'function'

    suite.export(module)
