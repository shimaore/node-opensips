OpenSIPS MI tooling
===================

    class OpensipsTimeoutError extends Error
    class OpensipsStatusError extends Error
    class OpensipsSyntaxError extends Error

    module.exports =

Send a command to MI using UDP: `mi_udp`
----------------------------------------

      mi_udp: (host = '127.0.0.1',port,command) ->

Looking at the `mi_datagram` code source, it looks like the response must fit in a single datagram.
If this isn't the case then this code needs to be rewritten.

        new Promise (resolve,reject) ->
          try
            client = dgram.createSocket 'udp4'

            client.on 'error', (error) ->
              reject error

            response = new Buffer 0

It is an error if our request receives no response packet.

            initial_timeout = ->
              client.close()
              reject new OpensipsTimeoutError 'timeout'

However since UDP has no way to indicate end-of-stream, simply assume we're done if we are no longer receiving responses.

            on_timeout = ->
              client.close()
              resolve response

            timeout = null

            client.on 'message', (msg,rinfo) ->
              buffer = Buffer.concat [response, msg]

              clearTimeout timeout
              timeout = setTimeout on_timeout, 2000

            message = new Buffer(command)
            client.sendAsync message, 0, message.length, port, host
            .then ->
              timeout = setTimeout initial_timeout, 2000

          catch error
            reject error

Convert a command and its arguments into a single string command
----------------------------------------------------------------

Supports both named and positional arguments.

      command: (command,args...) ->
        make_value =  (x) ->
          if not x?
            return ''
          x = ''+x
          if x.match /['"\n\r]/
            '"'+x+'"'
          else
            x

        if args.length is 1 and typeof args[0] is 'object'
          # Named arguments
          o = args[0]
          arg_values = ("#{k}::#{make_value v}" for k,v of o)
        else
          # Positional arguments
          arg_values = args.map(make_value)

        [":#{command}:",arg_values...].map( (x) -> x+"\n" ).join ''

Parse a response (a Buffer instance or a string)
------------------------------------------------

Upon success, the `body` field of the result is an Array which might contain `value` fields beyond the numbered elements.
Will `throw` on non-successful status or syntax error.

      parse: (response) ->
        if typeof response isnt 'string'
          response = response.toString 'utf-8'

The first line is a status line, while any remaining lines should contain data.

        [status,lines...] = response.split /\n/

First handle errors. We might for example get '404 AOR not found' or some other message.

        if status isnt '200 OK'
          throw new OpensipsStatusError status

        body = []
        path = []

        body_ref = (p,b = body) ->
          if p.length is 0
            return b

          name = p[0]
          b[name] ?= []
          body_ref p[1..], b[name]

The body may contain any number of (key,value) entries or (key,hash) entries.

        for line in lines when line isnt ''

          m = line.match /^(\t*)([^\t]*)$/
          if not m
            throw new OpensipsSyntaxError "Invalid data line: #{line}"

          depth = m[1].length

In case of un-indent, retrieve the proper (shorter) path.

          while path.length > depth
            path.pop()

Parse the new content

          n = m[2].match /^(\w+)::\s*(.*)$/
          if n
            key = n[1]
            value = n[2]
          else
            key = null
            value = m[2]

          if value?
            ref = body_ref path
            if key?
              ref[key] ?= []
              o = []
              o.value = value
              ref[key].push o
            else
              ref.push value

          if key?
            path.push key

        return body

      OpensipsTimeoutError: OpensipsTimeoutError
      OpensipsStatusError: OpensipsStatusError
      OpensipsSyntaxError: OpensipsSyntaxError

    Promise = require 'bluebird'
    dgram = Promise.promisifyAll require 'dgram'
