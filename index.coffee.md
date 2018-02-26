JSON-path manipulators
----------------------

This library is meant to be used inside Node.js or CouchDB `_update` functions etc. to update a document using a series of commands.

    Immutable = require 'immutable'

The semantics of the operations are that no changes are made if the changes are invalid.

    operations =

Assign

      set: (doc,name,args) ->
        args = Immutable.List args
        return doc unless args.size is 1
        doc.setIn name, args.get 0

Remove

      remove: (doc,name) ->
        doc.removeIn name

Set difference

      minus: (doc,name,args) ->
        return doc unless doc.hasIn name
        value = doc.getIn name
        return doc unless Immutable.Set.isSet value
        args = Immutable.Set args
        doc.setIn name, value.subtract args

Set union

      union: (doc,name,args) ->
        return doc unless doc.hasIn name
        value = doc.getIn name
        return doc unless Immutable.Set.isSet value
        args = Immutable.Set args
        doc.setIn name, value.union args

List append

      append: (doc,name,args) ->
        return doc unless doc.hasIn name
        value = doc.getIn name
        return doc unless Immutable.List.isList value
        args = Immutable.List args
        doc.setIn name, value.concat args

List splice

      splice: (doc,name,args) ->
        return doc unless doc.hasIn name
        value = doc.getIn name
        return doc unless Immutable.List.isList value

        args = Immutable.List args
        return doc unless 0 < args.size <= 3
        start = args.get 0
        count = args.get 1
        rep = args.skip 2
        return doc unless typeof start is 'number'
        return doc unless typeof count is 'number'

        doc.setIn name, value.splice start,count,rep...

    default_forbidden_paths = Immutable.Set [
      '_id'
      '_rev'
      '_deleted'
    ].map (x) -> Immutable.List.of x

    parse = require './parse'

    module.exports = json_path = (doc,path,operation,args,forbidden_paths = default_forbidden_paths) ->
      op = operations[operation]
      unless op?
        console.error "Unknown operation #{operation}"
        return doc

      assert Immutable.Map.isMap doc
      assert Immutable.List.isList args

      if typeof path is 'string'
        path = Immutable.List parse path
      else
        path = Immutable.List path

      return doc if forbidden_paths.has path
      op doc, path, args

    assert = require 'assert'
