**Table of Contents**  *generated with [DocToc](http://doctoc.herokuapp.com/)*

- [Data](#data)
- [Mini library](#mini-library)
	- [loadImages()](#loadimages)
	- [class MainTimer](#class-maintimer)
	- [class Timer](#class-timer)
	- [getRandomInt()](#getrandomint)
	- [choose()](#choose)
	- [removeFromArray()](#removefromarray)
	- [c()](#c)
	- [t()](#t)
	- [jV()](#jv)
	- [include()](#include)
	- [class CircularArray](#class-circulararray)
- [Resources](#resources)
- [GUI](#gui)
	- [Selectable](#selectable)
	- [class TextChooser](#class-textchooser)
	- [class NumberChanger](#class-numberchanger)
	- [class Activateable](#class-activateable)
	- [class ColorBar](#class-colorbar)
	- [class PropertiesDisplay](#class-propertiesdisplay)
	- [class ShipEditor](#class-shipeditor)
	- [class KeyChanger](#class-keychanger)
	- [class Menu](#class-menu)
- [Other classes](#other-classes)
	- [class ComputerPlayer](#class-computerplayer)
	- [class InputState](#class-inputstate)
- [Game states](#game-states)
	- [class Fight](#class-fight)
	- [class Lobby](#class-lobby)
- [Entities](#entities)
	- [class Ship](#class-ship)
	- [class Shoot](#class-shoot)
	- [class ImageAnimation](#class-imageanimation)
- [Main routine](#main-routine)

## Data

The object DATA contains

* presets for ship configurations.
* the maximum numbers for the ship values
* keyCode for WASD and arrows
* game dimensions
* and others like CSS code to flip an image in all browsers

<!--hack-->

    DATA =
      energyMax : 1000
      damageMax : 100
      rofMax : 100
      speedMax : 20
      presets : [
        presetName:'normal'
        shipName:'hunter'
        properties:
          energy:50
          damage:5
          
The property `rof` means: Rate of fire. A lower value makes the ship shoot 
faster.

          rof:15
          speed:8
          
The comma is at a weird location. It's one column before `presetName`. 
It's necessary to tell the CoffeeScript-Compiler a new object begins.

       ,
        presetName:'light'
        shipName:'scout'
        properties:
          energy:35
          damage:2
          rof:5
          speed:10
       ,
        presetName:'heavy'
        shipName:'destroyer'
        properties:
          energy:75
          damage:20
          rof:60
          speed:5
      ]
      action_keyCode_maps: [
        up: 87
        down:83
        right:68
        left:65
       ,
        up: 38
        down:40
        right:39
        left:37
      ]
      areaHeight: 500
      areaWidth: 1000

The property `shipAreaGap` indicates the gap to the end of the field.

      shipAreaGap: 5
      
The property mode is set to *online* if the connection to the server 
with socket.io succeeds.

      mode:'offline'
      CSS:
        'flip-horizontal':
          '-moz-transform': 'scaleX(-1)'
          '-webkit-transform': 'scaleX(-1)'
          'transform': 'scaleX(-1)'
          'filter': 'fliph'
          '-o-transform': 'scaleX(-1)'

## Mini library

### loadImages()

The function `loadImages` preloads images.

Parameters:

* folder : The name of the folder that is in the folder *images*.
The folder contains all images for an animation.
* number : Number of images in the folder.
* width : Width of the images
* height : Height of the images

The images must have the following naming convention: *img ( i ).png*, where i 
is a number beginning from 1. Win7 renames several files together like this.

    loadImages = (folder, number, width, height) ->
      images = new Array
      images.width = width
      images.height = height
      for i in [1..number]
        img = new Image
        img.src = 'images/' + folder + '/img (' + i + ').png'
        images.push img
      images

### class MainTimer

I'm sure there are already libraries out there which try the same. You can add 
several `Timer` to an instance of `MainTimer`. `MainTimer` can be stopped and 
so will all attached timers. The class `MainTimer` expects the parameter 
interval. A number in milliseconds. The default value is 10, because I think 
that is what most environments can handle. However, I will set the param 
interval to a higher number.

    class MainTimer #meta
      constructor: (interval = 10) ->
        @interval = interval #ms
        @running = false
        @timers = []
        @intervalId = undefined
      start: ->
        @stop()
        @intervalId = window.setInterval((=> @tick()), @interval)
        @running = true
      stop: ->
        window.clearInterval(@intervalId)
        @running = false
      tick: ->
        for timer in @timers
          if timer? and !timer.stop
            timer.count += 1
            if timer.count >= timer.interval
              timer.fn()
              if timer.repeat
                timer.count = 0
              else
                removeFromArray(timer, @timers)
      add:(timer) ->
        @timers.push timer
      remove:(timer) ->
        removeFromArray(timer, @timers)

### class Timer

Instances of `Timer` are attached to `MainTimer`. The variable `count` is 
increased by 1 whenever `tick` of `MainTimer` is called.

Parameters:

* fn : The function that will be called when the variable `count` meets 
`interval`.
* interval : A higher value implies a longer time.
* repeat : If `fn` is called and `repeat` is true, `count` is set to 0. 
Otherwise, the `Timer` is removed from `MainTimer`.

The variable `stop` can be set to true. Like this, the timer stays in 
`MainTimer`, but `count` won't be increased until you set `stop` back 
to `true`.

    class Timer #meta
      constructor:(@fn,@interval,@repeat = true,@count = 0) ->
        @stop = false

### getRandomInt()

Simple function to get a random integer. 

    getRandomInt = (min, max) ->
      Math.floor(Math.random() * (max - min + 1)) + min

### choose()
The function `choose` uses the previous function `getRandomInt` to choose 
randomly between several elements.

    choose = (elements) ->
      elements[getRandomInt(0, elements.length - 1)]

### removeFromArray()

Not all browsers support `indexOf`, that's why I use `inArray` from JQuery. 
JQuery is anyway used in this project. `removeFromArray` removes an element from 
an array.

    removeFromArray = (element, array) ->
      index = $.inArray(element,array)
      if index > -1
        array.splice(index,1)
        
### c()

The function `c` is used to create DOM nodes and to attach them to a parent. 
The aim is to simplify DOM creation. JQuery offers `.html` to create DOM trees 
out of html, but I don't like writing html because of the necessary closing 
tags.

* tag : a tag name or a DOM - Element 
* children : DOM - Elements or objects with `dom` property..

<!--hack-->

    c = (tag,children...) ->
      if tag.nodeName then el = tag
      else el = document.createElement tag
      for child in children
        if child.dom then child = child.dom
        el.appendChild child
      el        

### t()

Creates a text node.

    t = (text) ->
      document.createTextNode(text)
        
### jV()

`jV` means *JSON variables*. I wanted the possibility to add variables as keys. 
The parameter `data` is an array. The first element is the key, the second the 
value etc.

    jV : (data) ->
        object = {}
        for el,i in data by 2
            key = el
            value = data[i+1]
            object[key] = value
        object        

### include()

`include` adds methods or properties to a class. Use it as first statement 
in the class body.

Parameters:

* target : The class to extend, mostly *this*
* obj : An object containing methods or properties to add.

<!--hack-->

    include = (target,obj) -> #integrates Mixin
      for key, value of obj
        target::[key] = value

### class CircularArray

It's an array with an index that can be de/in-creased. When it reaches its 
maximum, the index continues at 0. `get` returns the current element or the 
element with the specified index. `îndex` returns the current index or sets 
it.

    class CircularArray #meta
      constructor:(@array = []) ->
        @i = 0
      next: ->
        @i = (@i+1) % @array.length
        @
      previous: ->
        if (@i-=1) < 0 then @i=(@array.length - 1)
        @
      get:(i) ->
        @array[i ? @i]
      index:(i) ->
        if i?
          @i = i
        @i
      add:(el) ->
        @array.push(el)
        el
      remove:(el) ->
        removeFromArray(el, @array)
        el
      addAfter:(i,el) ->
        start = @array.slice(0,i+1)
        end = @array.slice(i+1)
        start.push el
        @array = start.concat end
      addBefore:(i,el) ->
        a = @array.slice(0,i)
        a.push el
        @array = a.concat @array.slice(i)

## Resources

The object `RES` contains data available to all functions or classes in this 
module. It's a kind of global storage.

    RES =
      images:
        explosion: loadImages('explosion', 8, 32, 32)
        countdown: loadImages('countdown', 24, 320, 180)
      mainTimer: new MainTimer 40
      socket: undefined

## GUI

GUI - Objects have the property `dom`. `dom` stores the DOM - Element that 
can be added to the existing DOM tree.

### Selectable

`Selectable` is a *Mixin*. It can be added to a class to make it selectable.

    Selectable = #MIXIN
      select: ->
        $(@dom).css
          background:'black'
          color:'white'
      deselect : ->
        $(@dom).css
          background:''
          color:'black'

### class TextChooser
          
* Use `setTexts` to set the texts you could choose
* Use `setActions` to set the actions that occur when a text is chosen
* Use `setData` to set the text to the given one. Text must have been set by 
`setTexts`.

<!--hack-->

    class TextChooser #primitive GUI element
      include(@,Selectable)
      constructor: ->
        @dom = c 'span',
          @txt = t ''

      next : ->
        @activate(@texts.next().index())
      previous: ->
        @activate(@texts.previous().index())
      activate: (index) ->
        @txt.data = @texts.get(index)
        @actions?[index]?()
      setData: (data) ->
        i = $.inArray(data, @texts.array)

        if i > -1
          @texts.index(i)
          @txt.data = data
      setTexts:(texts) ->
        @texts = new CircularArray(texts)
      setActions: (@actions) ->

### class NumberChanger

Shows a number. The number can be changed. Use max, min and step.

    class NumberChanger #primitive GUI element
      include(@,Selectable)
      constructor:(@max,@min = 0,@step = 1) ->
        @dom = c 'span',
          @txt = t ''
      next: ->
        @txt.data = @number = Math.min(@number+@step,@max)
      previous: ->
        @txt.data = @number = Math.max(@number-@step,@min)
      setData:(data) ->
        @txt.data = @number = data
      getData: ->
        @number

### class Activateable

Shows a text, calls a function when activated.

    class Activateable #primitive GUI element
      include(@,Selectable)
      constructor:(@text, @action) ->
        @dom = c 'span',
          @txt = t @text
      next : ->
        @action?()

### class ColorBar

Used to display the health of a ship. The parameter `i` determines the 
direction in which the bar shrinks.

    class ColorBar #primitive GUI element
      constructor:(@width,@i) ->
        @dom = c 'div',
          @bar = c 'div'

        $(@dom).css
          width:@width + 'px'

        $(@bar).css
          height: '20px'
          position:'absolute'
          
        if @i is 1
          @bar.style.right = '0px'
      update:(n,maxn) ->
        if n < 0 then n = 0
        p = n/maxn
        f = parseInt(510*p)
        if p is 0
          color = 'rgb(255,0,0)'
        else if f <= 255
          color = 'rgb(255,' + f + ',0)'
        else if p is 1
          color = 'rgb(0,255,0)'
        else if f > 255
          f = f - 255
          f = 255 - f
          color = 'rgb(' + f + ',255,0)'
        @bar.style.background = color
        @bar.style.width = parseInt(@width * p) + 'px'

### class PropertiesDisplay

The `PropertiesDisplay` is used by `ShipEditor` to display the properties of a 
ship.

    class PropertiesDisplay
      constructor: ->
        @dom = c 'div'
        
Use `add` to add more properties.

      add: (property) ->
        @[property] = {}
        
Don't forget to add a limit for each property. 
Add a property in `DATA` with the form: 
*propertyNameMax*. Example: *damageMax*.

        max = DATA[property+'Max']
        if max >= 100 
          if max > 500 then step = 10 else step = 2
        else
          step = 1
        @dom.appendChild c 'div',
          @[property].image = img = c 'img'
          @[property].txt = new NumberChanger(max,0,step)
        img.src = 'images/'+property+'.png'
        $(img).css
          verticalAlign:'middle'
          margin:'4px'
          marginRight:'8px'
        @[property]
    
### class ShipEditor

You can change the values of a ship with the `ShipEditor`. 

    class ShipEditor #GUI element
      constructor: (@i, @action_keyCode_map) ->

First, we define handlers to control the GUI.

        
        cpuHandler = =>
          @cpu = true
          @end()
      
        @keyHandler = (e) =>
          switch e.which
            when @action_keyCode_map.down
              s.get().deselect()
              s.next()
              s.get().select()
            when @action_keyCode_map.up
              s.get().deselect()
              s.previous()
              s.get().select()
            when @action_keyCode_map.right
              s.get().next?()
            when @action_keyCode_map.left
              s.get().previous?()

Then we build the DOM tree. GUI Elements can be assigned to variables.

        @dom = c 'div',
          c 'div',
            @presetName = new TextChooser
          @imageContainer = c 'div',
            @image = c 'img'
            shipNameDiv = c 'div',
              @shipName = new TextChooser
          @properties = new PropertiesDisplay
          OKCont = c 'div',
            @CPU = new Activateable 'CPU', cpuHandler 
            c 'br'
            t ' '
            @OK = new Activateable('OK',@end)

Set CSS.

        if @i is 1
          $(@image).css DATA.CSS['flip-horizontal']
          
        $(@dom).css
          background:'url(images/grey.png)'
          # border:'1px solid black'
          position:'absolute'
          padding:'5px'
          width:'250px'
        $(@imageContainer).css
          border:'1px dashed black'
          background:'url(images/grey.png)'
          overflow:'hidden'
          minHeight:'80px'
          position:'relative'
          margin:'20px'
          padding:'10px'
          position:'relative'
        $(shipNameDiv).css
          left:'5px'
          bottom:'2px'
          position:'absolute'
          background:'url(images/grey.png)'
        $(OKCont).css
          textAlign:'right'
        $(@image).css
          position:'absolute'
        $(@dom).hide()

Set which one are selectable.

        @selectables = s = new CircularArray [@presetName,@shipName,@CPU,@OK]
        @selectables.get().select()

There are four methods to initialize a GUI, hide it, show it again and destroy 
it. Initialize a GUI with `start`. All event handler are attached. Destroy a 
GUI with `end`. Event handlers are removed. There is the possibility to pause a 
GUI to show another. Use `_break` and `_continue` for this.

      _continue: ->
        if @started
          $(@dom).show()
          $(document).on('keydown',@keyHandler)
      _break: =>
        if @started
          $(@dom).hide()
          $(document).off('keydown',@keyHandler)
      start:(@callback) ->
        @started = true
        @cpu = false
        $(@dom).show()
        $(document).on('keydown',@keyHandler)
      end: =>
        @started = false
        $(@dom).hide()
        $(document).off('keydown',@keyHandler)
        @callback(@getData())
        
Set data with `setData`. `data` is a list of presets.

      setData:(data) ->
        @data = data
        @images = []

It extracts all the preset names.

        presetNames = (preset.presetName for preset in data)
        @presetName.setTexts presetNames
        actions = []
        
Look at the double dashed arrows. Two of them! Otherwise 
the variable `i` isn't unique for each action.

        for name, i in presetNames
          actions.push ((v) => => @setPreset v)(i)
        @presetName.setActions actions

It extracts all the ship names.

        shipNames = (preset.shipName for preset in data)
        @shipName.setTexts shipNames
        actions = []
        for name,i in shipNames
          img = new Image
          img.src = 'images/' + name + '.png'
          @images.push img

          actions.push ((index) => =>
            @image.src = @images[index].src
            @image.onload = => @centerImage()
          )(i) #weird workaround
        @shipName.setActions actions

        for property of data[0].properties
          @properties.add(property)
          @selectables.addBefore(@selectables.array.length-2,
            @properties[property].txt)

        @setPreset(0)

Set now one of the presets to display.

      setPreset:(i) ->
        preset = @data[i]
        @presetName.setData(preset.presetName)
        @image.src = @images[i].src
        @image.onload =  => @centerImage()
        @shipName.setData(preset.shipName)
        for property, value of preset.properties
          @properties[property].txt.setData(value)

After a ship is chosen you can get the values with `getData`.

      getData: ->
        data = {}
        data.presetName = @presetName.texts.get()
        data.shipName = @shipName.texts.get()
        for property of @data[0].properties
          data[property] = @properties[property].txt.getData()
        data.cpu = @cpu
        data
        
`centerImage` centers the image within `imageContainer` container.

      centerImage: ->
        @image.style.left =  (($(@imageContainer).innerWidth()/2) -
          (@image.width/2)) + 'px'
        @image.style.top = (($(@imageContainer).innerHeight()/2) -
          (@image.height/2)) + 'px'

### class KeyChanger

    class KeyChanger #GUI element
      constructor: ->
        @dom = c 'div'
        $(@dom).css
          position:'absolute'
          textAlign:'center'
          left:'50%'
          top:'50%'
          width:'500px'
          marginLeft:'-250px'
          border:'1px dashed black'
          background:'url(images/grey.png)'

        $(@dom).hide()
      setKey: ->
        $(@dom).text 'Push a key for \"'+@names[@i]+'\"'
        $(document).one('keydown', ((e) =>
          @nameKeyPairs[@names[@i]] = e.which
          @i++
          if @i is @names.length
            @end()
          else
            @setKey()
        ))
      start:(@nameKeyPairs,@callback) ->
        @names = (name for name of nameKeyPairs)
        @i = 0
        @setKey()
        $(@dom).show()
      end: ->
        $(@dom).hide()
        @callback()

### class Menu

    class Menu #GUI element
      constructor: ->
        @dom = c 'div'
        $(@dom).css
          width:'500px'
          position:'absolute'
          left:'50%'
          top:'50px'
          marginLeft:'-250px'
          textAlign:'center'
          background:'url(images/grey.png)'
        $(@dom).hide()

        @keyHandlerMenu = (e) =>
          if e.which is 80 #p
            @end()

        c1 = DATA.action_keyCode_maps[0]
        c2 = DATA.action_keyCode_maps[1]
        @keyHandler = (e) =>
          switch e.which
            when c1.down,c2.down
              @selectables.get().deselect()
              @selectables.next()
              @selectables.get().select()
            when c1.up,c2.up
              @selectables.get().deselect()
              @selectables.previous()
              @selectables.get().select()
            when c1.right,c2.right
              @selectables.get().next?()
            when c1.left,c2.left
              @selectables.get().previous?()
        @_continue = =>
          $(@dom).show()
          $(document).on('keydown',@keyHandler)
          $(document).on('keydown',@keyHandlerMenu)
        @start = (@callback) =>
          @_continue()
        @_break = =>
          $(@dom).hide()
          $(document).off('keydown',@keyHandler)
          $(document).off('keydown',@keyHandlerMenu)
        @end = =>
          @_break()
          @callback()
      setData:(@texts, @actions) ->
        $(@dom).empty()
        @activateables = []

        for i in [0...@texts.length]
          @activateables.push(new Activateable(@texts[i], @actions?[i]))
          $(@dom).append @activateables[i].dom
          $(@dom).append c 'br'
          $(@dom).append c 'br'

        @selectables = new CircularArray @activateables
        @selectables.get().select()

## Other classes

### class ComputerPlayer

    class ComputerPlayer #meta
      constructor: (@ship) ->
        @enemy = @ship.enemy
        @i = 0
        @mode = ''
        @backToCenterModeLength = 25
        @timer = new Timer @tick, 1
      tick:=> #add to timer
        @ship.inputState.right = true

        if @i > 0
          @i-=1
          @[@mode]()
        else
          if @isInDanger()
            @avoid()
          else
            @hunt()
      isInDanger: ->
        for shoot in @enemy.shoots
          if shoot.y + Shoot.size > @ship.y and shoot.y < @ship.y + @ship.height
            return true
        return false
      hunt: ->
        enemyMiddle = @enemy.y + (@enemy.height/2)
        shipMiddle = @ship.y + (@ship.height/2)
        if shipMiddle > enemyMiddle  + 20 then @up()
        else if shipMiddle < enemyMiddle  - 20 then @down()
        else @stop()
      avoid: ->
        shoot = @enemy.shoots[0]
        shootMiddle = shoot.y + (Shoot.size/2)
        shipMiddle = @ship.y + (@ship.height/2)

        if  shootMiddle < shipMiddle
          if @isPossibleMove('down') 
            @i = 15
            @mode = 'down'
          else
            @backToCenterMode('up')
        else if shootMiddle > shipMiddle
          if @isPossibleMove('up') 
            @i = 15
            @mode = 'up'
          else
            @backToCenterMode('down')
        else
          choose([@up, @down])()
      isPossibleMove: (move) ->
        switch move
          when 'up'
            return @ship.y - @ship.data.speed > 0
          when 'down'
            return @ship.y + @ship.height + @ship.data.speed < DATA.areaHeight
      backToCenterMode: (mode) ->
        @i = @backToCenterModeLength
        @mode = mode
      stop: =>
        @ship.inputState.up = false
        @ship.inputState.down = false
        @ship.inputState.last = ''
      up: =>
        @ship.inputState.up = true
        @ship.inputState.down = false
        @ship.inputState.last = 'up'
      down: =>
        @ship.inputState.up = false
        @ship.inputState.down = true
        @ship.inputState.last = 'down'

### class InputState

    class InputState #meta
      constructor:(@action_keyCode_map) ->
        @keyCode_action_map = {}
        for action, keyCode of @action_keyCode_map
          @[action] = false
          @keyCode_action_map[keyCode] = action
      keyEvent : (e,isDown) =>
      
        keyCode = e.which
        action = @keyCode_action_map[keyCode]
        if action?
          @[action] = isDown
          @last = action
      eventHandlerDown:(e) =>
        @keyEvent(e,true)
      eventHandlerUp:(e) =>
        @keyEvent(e,false)
      addHandler: ->
        $(document).on('keydown',@eventHandlerDown )
        $(document).on('keyup',@eventHandlerUp)
      removeHandler: ->
        $(document).off('keydown',@eventHandlerDown)
        $(document).off('keyup',@eventHandlerUp)
        @allFalse()
      allFalse: ->
        for action of @action_keyCode_map
          @[action] = false
        @last = ''

## Game states

### class Fight

    class Fight #game state
      constructor: (@ships,@menu,@ani) ->
        @waitingForShoots = false
        @computerPlayers = []
        for ship in @ships
          @computerPlayers.push new ComputerPlayer ship
          
      keyHandlerMenu: (e) =>
        if e.which is 80 #p
          @_break()
          @menu.start =>
            @ani.play =>
              @_continue()

      waitForShoots: (e) =>
        if !@waitingForShoots
          @waitingForShoots = true
          
          for ship in @ships
            ship.removeHandler()
          
          destroyedShip = e.target 
          if destroyedShip.shoots.length > 0
            $(destroyedShip).one 'noShoots', =>
              @end()
          else @end()
        
      socketTickListener: (@inputState0,@inputState1) =>
        RES.mainTimer.tick()
        
      start: (@callback) ->
        for ship, i in @ships when ship.data.cpu
          RES.mainTimer.add @computerPlayers[i].timer
          
      
        RES.mainTimer.start()

        for ship in @ships
          $(ship).one 'destroyed', @waitForShoots

        if DATA.mode is 'online' then RES.socket.on 'tick', @socketTickListener

        $(document).on('keydown', @keyHandlerMenu)
        
      end: =>
        for computerPlayer in @computerPlayers
          RES.mainTimer.remove computerPlayer.timer
        for ship in @ships
          ship.removeHandler()
          
        for ship in @ships
          shoots = ship.shoots.slice()
          for shoot in shoots
            shoot?.remove()

        @waitingForShoots = false
        
        RES.mainTimer.stop()

        RES.socket?.removeListener 'tick', @socketTickListener

        $(document).off('keydown', @keyHandlerMenu)
        
        for ship in @ships
          $(ship).off 'destroyed', @waitForShoots
        
        @ships[0].destroyed = false
        @ships[1].destroyed = false

        setTimeout((=> @callback()), 500)
      _break: ->
        $(document).off('keydown', @keyHandlerMenu)
        if DATA.mode is 'offline'
          RES.mainTimer.stop()
        else
          RES.socket?.removeListener 'tick', @socketTickListener
      _continue: ->
        $(document).on('keydown', @keyHandlerMenu)
        if DATA.mode is 'offline'
          RES.mainTimer.start()
        else if DATA.mode is 'online'
          RES.socket.on 'tick', @socketTickListener

### class Lobby

    class Lobby #game state
      constructor:(@menu,@ships,@editors) ->
      
      keyHandlerMenu:(e) =>
        if e.which is 80 #p
          @_break()
          @menu.start =>
            @_continue()
            
      start: (@callback) ->
        ready = 0

        for ship, i in @ships
          $(ship.dom).hide()
          $(ship.bar.dom).hide()
          @editors[i].start ((index) => (data) =>
            editedShip = @ships[index]
            editedShip.setData data
            $(editedShip.dom).show()
            $(editedShip.bar.dom).show()
            ready += 1
            if ready is @ships.length then @end()
          )(i)

        $(document).on('keydown', @keyHandlerMenu)
      end:=>
        $(document).off('keydown', @keyHandlerMenu)
        @callback()
      _break:=>
        $(document).off('keydown', @keyHandlerMenu)
        for editor in @editors
          editor._break()
      _continue:=>
        $(document).on('keydown', @keyHandlerMenu)
        for editor in @editors
          editor._continue()
      
## Entities
      
### class Ship

    class Ship #entity
      constructor: (@i) ->
        #DOMF
        @dom =  c 'img'
        @bar = new ColorBar 300, @i
        if @i is 1
          $(@dom).css DATA.CSS['flip-horizontal']

        @shoots = []
        @laserLoading = false
        
        @inputState = new InputState DATA.action_keyCode_maps[@i]
        
        @handler = =>
          if @inputState.up and !@inputState.down
            @up()
          else if !@inputState.up and @inputState.down
            @down()
          else if @inputState.up and @inputState.down
            if @inputState.last is 'up'
              @up()
            else if @inputState.last is 'down'
              @down()
          if @inputState.right
            @shoot()
            
        @timer = new Timer(@handler, 1)
          
      addHandler: ->
        @inputState.addHandler()
        RES.mainTimer.add @timer
      removeHandler: ->
        @inputState.removeHandler()
        RES.mainTimer.remove @timer
      setY: (y) ->
        @y = y
        @dom.style.top = @y + 'px'
      up: ->
        @setY Math.max(@y - @data.speed, 0)
      down: ->
        @setY Math.min(@y + @data.speed, DATA.areaHeight - @height)
      shoot: ->
        if !@laserLoading
          shoot = new Shoot(@,@i)
          @shoots.push shoot
          $(shoot).one 'destroyed', =>
            removeFromArray(shoot,@shoots)
            if @shoots.length is 0 then $(@).trigger('noShoots')
          @laserLoading = true
          RES.mainTimer.add new Timer((=>@laserLoading = false), @data.rof, false)
      takeDamage: (damage) ->
        @energy -= damage
        @bar.update(@energy,@data.energy)
        if @energy <= 0
          if !@destroyed then @explode()
      explode: ->
        @inputState.removeHandler()
        @inputState.allFalse()
        
        
        @destroyed = true
        $(@).trigger('destroyed')
        $(@dom).hide()

        ani = new ImageAnimation(RES.images.explosion,4,8,true)
        $(ani.dom).css
          position:'absolute'
          top: (@y + (@height/2) - (ani.dom.height/2)) + 'px'
          left: (@x + (@width/2) - (ani.dom.width/2)) + 'px'
        $(@dom.parentNode).prepend ani.dom
        ani.play =>
          @dom.parentNode.removeChild ani.dom
      setData:(@data) ->
        @dom.src = 'images/' + @data.shipName + '.png'
        @dom.onload = =>
          @width = @dom.width
          @height = @dom.height
          
          $(@dom).css
            top:(@y = (DATA.areaHeight/2) - (@height/2)) + 'px'
            position:'absolute'
          if @i is 0
            $(@dom).css
              left:(@x = DATA.shipAreaGap) + 'px'
          else
              $(@dom).css
                left:(@x = DATA.areaWidth - DATA.shipAreaGap - @width) + 'px'
            
            
        @energy = @data.energy
        @bar.update(@energy, @data.energy)

### class Shoot

    class Shoot #entity
      @speed = 40
      @size = 5
      constructor:(@ship,@i) ->
        #DOM
        @dom = c 'div'
        $(@dom).css
          width: Shoot.size + 'px'
          height: Shoot.size + 'px'
          position:'absolute'
          background:'black'

        if @i is 0
          @direction = 1
          @setX(@ship.width + DATA.shipAreaGap)
          @timer = new Timer(@moveRight,1)
        else
          @direction = -1
          @setX(DATA.areaWidth - DATA.shipAreaGap - @ship.width - Shoot.size)
          @timer = new Timer(@moveLeft,1)

        @y = @ship.y + (@ship.height/2) - (Shoot.size/2)
        @dom.style.top = @y + 'px'

        @ship.dom.parentNode.appendChild @dom

        RES.mainTimer.add @timer
      setX: (x) ->
        @x = x
        @dom.style.left = x + 'px'
      moveRight: =>
        @setX(@x += Shoot.speed)

        if @x > DATA.areaWidth
          @remove()
        else
          if @isCollisionRight()
            @ship.enemy.takeDamage(@ship.data.damage)
            @remove()
      moveLeft: =>
        @setX(@x -= Shoot.speed)

        if @x < -Shoot.size
          @remove()
        else
          if @isCollisionLeft()
            @ship.enemy.takeDamage(@ship.data.damage)
            @remove()
      isCollisionRight: ->
        if (@x + Shoot.size) > (DATA.areaWidth - @ship.enemy.width - Shoot.size)
          if (@y + Shoot.size) > (@ship.enemy.y) and 
              @y < (@ship.enemy.y + @ship.enemy.height)
            return true
        return false
      isCollisionLeft: ->
        if (@x) < (@ship.enemy.width + Shoot.size)
          if (@y + Shoot.size) > (@ship.enemy.y) and 
              @y < (@ship.enemy.y + @ship.enemy.height)
            
            return true
        return false
      remove: ->
        if $.contains(@ship.dom.parentNode, @dom) then @ship.dom.parentNode.removeChild @dom
        RES.mainTimer.remove @timer
        $(@).trigger('destroyed')
        delete @
    
### class ImageAnimation

    class ImageAnimation #entity
      constructor:(@images,@speed,fnAt,@ownTimer,
          @repeat = false) ->

        @fnAt = fnAt ? @images.length

        @dom = c 'img'
        @dom.src = @images[0].src

        @i = 1
        @timer = new Timer((=>
          if @i?
            @i += 1
            @dom.src = @images[@i-1].src
            if @i is @fnAt
              @callback()
            if @i is @images.length
              if @repeat then @i = 0
              else
                @stop()
          ), @speed)

        if @ownTimer
          @mainTimer = new MainTimer
          @mainTimer.add @timer

        $(@dom).hide()
      play:(@callback) ->
        $(@dom).show()
        if @ownTimer then @mainTimer.start() else RES.mainTimer.add(@timer)
      stop: ->
        @i = 0
        if @ownTimer then @mainTimer.stop() else RES.mainTimer.remove(@timer)
      pause: ->
        if @ownTimer then @mainTimer.stop() else RES.mainTimer.remove(@timer)

## Main routine

    $(document).ready ->
      # if io?
        # RES.socket = io.connect()
        
        # RES.socket.on 'connect', ->
          # DATA.mode = 'online'

          # getHandler_Out = (eventName) ->
            # (e) -> 
              # if !e.extern
                # RES.socket.emit(eventName, e.which)
                
          # getHandler_In = (eventName) ->
            # (keyCode) ->
              # e = $.Event(eventName)
              # e.which = Number(keyCode)
              # e.extern = true
              # $(document).trigger(e)

          # events = ['keydown','keyup']
          # for event in events
            # $(document).on event, getHandler_Out(event)
            # RES.socket.on event, getHandler_In(event)

      #initialize
      area = c 'div'
      $(area).css
        top: '50%'
        marginTop: -(DATA.areaHeight/2) + 'px'
        left: '50%'
        marginLeft: -(DATA.areaWidth/2) + 'px'
        # border: '1px solid black'
        background:'url(images/grey.png)'
        height: DATA.areaHeight
        width: DATA.areaWidth
        position:'fixed';
        overflow:'hidden'
        fontFamily:'"Lucida Console", Monaco, monospace'

      keyChanger = new KeyChanger
      
      action_keyCode_maps = DATA.action_keyCode_maps
      
      menu = new Menu 
      getHandler = (action_keyCode_map) ->
        ->
          menu._break()
          keyChanger.start action_keyCode_map, ->
            menu._continue()
      menu.setData ('change keys ' + i for map, i in action_keyCode_maps), 
        (getHandler(map) for map in action_keyCode_maps)
        
      players = 2
      ships = []
      editors = []
      
      directions = ['left', 'right']

      for player in [0...players]
        editors.push(editor = new ShipEditor player, action_keyCode_maps[player])
        editor.setData DATA.presets
        $editor = $(editor.dom)
        $editor.css directions[player], '20px'
        $editor.css 'top', '20px'
        
        ships.push(ship = new Ship player)
        
        $bar = $(ship.bar.dom)
        $bar.css
          position:'absolute'
          top:'5px'
        $bar.css directions[player], '5px'
      ships[0].enemy = ships[1]
      ships[1].enemy = ships[0]
          

      lobby = new Lobby menu, ships, editors
      
      ani = new ImageAnimation(RES.images.countdown,4,19,true)
      $(ani.dom).css
        position:'absolute'
        left:'50%'
        top:'50%'
        marginLeft: -(ani.images.width/2) + 'px'
        marginTop: -(ani.images.height/2) + 'px'
        
      fight = new Fight(ships,menu,ani)

      c area,
        keyChanger
        menu
        ships[0]
        ships[1]
        ships[0].bar
        ships[1].bar
        editors[0]
        editors[1]
        ani

      cycle = ->
        lobby.start ->
          for ship in ships
            ship.addHandler()
          ani.play ->
            if DATA.mode is 'online' then RES.socket.emit 'fight'
            fight.start ->
              cycle()
      cycle()
      
      $('body').append area
      $('body').css
        background:'url(images/grey.png)'
