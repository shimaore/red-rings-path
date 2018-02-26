    assert = require 'assert'
    describe 'The parser', ->
      it 'should parse properly', ->
        parse = require '../parse'
        assert.deepEqual ['hello','world'], parse 'hello.world'
        assert.deepEqual ['hello','world'], parse 'hello[world]'
        assert.deepEqual ['hello','world'], parse 'hello."world"'
        assert.deepEqual ['hello','world','3'], parse 'hello."world".3'
        assert.deepEqual ['hello','[world]'], parse 'hello."[world]"'
        assert.deepEqual ['hello','"world"'], parse 'hello.["world"]'
        assert.deepEqual ['àéï','world','ùù'], parse 'àéï"world"[ùù]'
