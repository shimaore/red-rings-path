    module.exports = (path) ->
      reg = /([^\.\"\[\]]+)\.?|\"([^"]+)\"\.?|\[([^\]]+)\]\.?/y
      reg.lastIndex = 0
      while $ = path.match reg
        $[1] ? $[2] ? $[3]
