Note that this test must be ran with a local instance of OpenSIPS running, listening on port 30010 for MI UDP commands.

    chai = require 'chai'
    chai.should()

    describe 'The MI interface', ->
      it 'should answer properly to `which`', ->
        {mi_udp,command,parse} = require '../mi'
        mi_udp '127.0.0.1', 30010, command 'which'
        .then (r) ->
          p = parse r
          p.should.be.instanceOf Array
          p.should.not.be.empty
          p.should.include.members ['which','help']
