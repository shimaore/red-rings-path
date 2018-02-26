    assert = require 'assert'
    Immutable = require 'immutable'

    describe 'The module', ->
      json_path = require '..'

      test = (doc,path,op,args...) ->
        reviver = (key,value) ->
          if Immutable.isKeyed value
            value.toMap()
          else
            if key is 's'
              value.toSet()
            else
              value.toList()

        from = (x) ->
          if x? and typeof x is 'object'
            Immutable.fromJS doc, reviver
          else
            x

        doc = json_path (from doc),path,op, Immutable.fromJS args
        doc.toJS()

      it 'should process changes', ->
        assert.deepEqual {a:3}, test {}, 'a', 'set', 3
        assert.deepEqual {a:[]}, test {}, 'a', 'set', []
        assert.deepEqual {a:[3]}, test {a:[]}, 'a', 'append', 3
        assert.deepEqual {s:[3,5]}, test {s:[3]}, 's', 'union', 5
        assert.deepEqual {s:[3,5]}, test {s:[3,5]}, 's', 'minus', 7
        assert.deepEqual {s:[3,5,{b:4}]}, test {s:[3,5]}, 's', 'union', b:4
        assert.deepEqual {s:[3,5]}, test {s:[3,5,{b:4}]}, 's', 'minus', b:4
        assert.deepEqual {a:[3,true]}, test {a:[3,5,{b:4}]}, 'a', 'splice', 1, 2, true
        assert.deepEqual {a:4}, test {}, 'a', 'set', 4
        assert.deepEqual {a:b:4}, test {}, 'a.b', 'set', 4
        assert.deepEqual {a:b:c:4}, test {}, 'a.b.c', 'set', 4
        assert.deepEqual {a:{}}, test {a:b:c:4}, 'a.b', 'remove'
        assert.deepEqual {a:b:c:4}, test {a:b:c:4}, 'a', 'set'
        assert.deepEqual {}, test {a:b:c:4}, 'a', 'remove'
        assert.deepEqual {_id:12}, test {_id:12}, '[_id]', 'set', 42
        assert.deepEqual {_id:12}, test {_id:12}, '"_deleted"', 'set', true
