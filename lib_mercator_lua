--module (..., package.seeall)

--[[
----------------------------------------------------------------
MERCATOR DSCHINI FOR CORONA SDK
----------------------------------------------------------------
PRODUCT  :		MERCATOR PROJECTION
VERSION  :		1.0.0
AUTHOR   :		MANFRED WEBER / DSCHINI.ORG
WEB SITE :		http://dschini.org
SUPPORT  :		manfred.weber@gmail.com
PUBLISHER:		DSCHINI.ORG
COPYRIGHT:		(C)2013 MANFRED WEBER & DSCHINI.ORG

----------------------------------------------------------------

PLEASE NOTE:
A LOT OF HARD AND HONEST WORK HAS BEEN SPENT
INTO THIS PROJECT AND WE'RE STILL WORKING HARD
TO IMPROVE IT FURTHER.
IF YOU DID NOT PURCHASE THIS SOURCE LEGALLY,
PLEASE ASK YOURSELF IF YOU DID THE RIGHT AND
GET A LEGAL COPY (YOU'LL BE ABLE TO RECEIVE
ALL FUTURE UPDATES FOR FREE THEN) TO HELP
US CONTINUING OUR WORK. THE PRICE IS REALLY
FAIR. THANKS FOR YOUR SUPPORT AND APPRECIATION.

FOR FEEDBACK & SUPPORT, PLEASE CONTACT:
MANFRED.WEBER@GMAIL.COM

]]--

require "sqlite3"

local isDevice = (system.getInfo("environment") == "device")

-- OBJECT TO HOLD LOCAL VARIABLES AND FUNCTIONS

local Mercator = {}
Mercator.FLAG_CITY          = 1
Mercator.FLAG_BIGCITY       = 2
Mercator.FLAG_FAMOUSCITY    = 4
Mercator.FLAG_CAPITALCITY   = 8
Mercator.debug              = true
Mercator.owner              = nil
Mercator.offset             = 268435456
Mercator.radius             = Mercator.offset/math.pi
Mercator.radiusKM           = 6371.0
Mercator.background         = nil
Mercator.projection         = display.newGroup()
Mercator.projection.dots    = {}
Mercator.drawings           = display.newGroup()
Mercator.controls           = display.newGroup()
Mercator.map                = nil
Mercator.borders            = nil
Mercator.mapHalf            = 480
Mercator.onEvent            = nil
Mercator.dbPath             = system.pathForFile("lib_dschini_mercator/mercator.db", system.ResourceDirectory)
Mercator.db                 = nil

function lengthOf( a, b )
    local width, height = b.x-a.x, b.y-a.y
    return (width*width + height*height)^0.5
end
local function calcAvgCentre( points )
    local x, y = 0, 0
    for i=1, #points do
        local pt = points[i]
        x = x + pt.x
        y = y + pt.y
    end
    return { x = x / #points, y = y / #points }
end
local function updateTracking( centre, points )
    for i=1, #points do
        local point = points[i]
        point.prevDistance = point.distance
        point.distance = lengthOf( centre, point )
    end
end
local function calcAverageScaling( points )
    local total = 0
    for i=1, #points do
        local point = points[i]
        total = total + point.distance / point.prevDistance
    end
    return total / #points
end

local queryPlaces = function(continents,countries,typebits,from,to,shuffle)
    local from = from or 0
    local to = to or 20
    local places = {}
    local sql = 'SELECT continents.rowid as continent_rowid,'
    sql = sql .. ' continents.code as continent_code,'
    sql = sql .. ' continents.label as continent_label,'
    sql = sql .. ' countries.rowid as country_rowid,'
    sql = sql .. ' countries.code as country_code,'
    sql = sql .. ' countries.label as country_label,'
    sql = sql .. ' places.rowid as place_rowid,'
    sql = sql .. ' places.label as place_label,'
    sql = sql .. ' places.latitude as place_latitude,'
    sql = sql .. ' places.longitude as place_longitude,'
    sql = sql .. ' places.typebit as place_typebit'
    sql = sql .. ' FROM continents,countries,places'
    sql = sql .. ' WHERE continents.rowid = countries.rowid_continent'
    sql = sql .. ' AND countries.rowid = places.rowid_country'
    if continents then
        sql = sql .. ' AND ( continents.code = "'
        sql = sql .. table.concat(continents,'" OR continents.code = "')
        sql = sql .. '" )'
    end
    if countries then
        sql = sql .. ' AND ( countries.code = "'
        sql = sql .. table.concat(countries,'" OR countries.code = "')
        sql = sql .. '" )'
    end
    if typebits then
        sql = sql .. ' AND ( places.typebit & '
        sql = sql .. table.concat(typebits,' != 0 AND places.typebit & ')
        sql = sql .. ' )'
    end
    if shuffle then
        sql = sql .. ' ORDER BY RANDOM()'
    end
    sql = sql .. ' LIMIT ' .. from .. ', ' .. to
    for row in Mercator.db:nrows(sql) do
        places[#places+1] =
        {
            continent_rowid = row.continent_rowid,
            continent_code = row.continent_code,
            continent_label = row.continent_label,
            country_rowid = row.country_rowid,
            country_code = row.country_code,
            country_label = row.country_label,
            place_rowid = row.place_rowid,
            place_label = row.place_label,
            place_latitude = row.place_latitude,
            place_longitude = row.place_longitude,
            place_typebit = row.place_typebit
        }
    end
    return places
end
Mercator.queryPlaces = queryPlaces

local LToX = function(x)
    return math.round(Mercator.offset+Mercator.radius*x*math.pi/180)
end

local LToY = function(y)
    return math.round(Mercator.offset-Mercator.radius*math.log((1+math.sin(y*math.pi/180))/(1-math.sin(y*math.pi/180)))/2)
end

local XToL = function(x)
    return ((math.round(x)-Mercator.offset)/Mercator.radius)*180/math.pi
end

local YToL = function(y)
    return (math.pi/2-2*math.atan(math.exp((math.round(y)-Mercator.offset)/Mercator.radius)))*180/math.pi
end

local DecimalDegreesToRadians = function(degrees)
    return degrees * math.pi / 180.0
end

local RadiansToDecimalDegrees = function(radians)
    return radians * 180.0 / math.pi
end

local getXFromLon = function( lon)
    return Mercator.mapHalf/Mercator.offset*(LToX( lon )- Mercator.offset)
end
Mercator.getXFromLon = getXFromLon

local getYFromLat = function( lat )
    return Mercator.mapHalf/Mercator.offset*(LToY( lat )- Mercator.offset) 
end
Mercator.getYFromLat = getYFromLat

local getLonFromX = function( x )
    return XToL( (x * (Mercator.offset/Mercator.mapHalf)) + Mercator.offset)
end
Mercator.getLonFromX = getLonFromX

local getLatFromY = function( y)
    return YToL( (y * (Mercator.offset/Mercator.mapHalf)) + Mercator.offset )
end
Mercator.getLatFromY = getLatFromY

local getDistance = function( lat1,lon1,lat2,lon2)
    local lat1 = DecimalDegreesToRadians(lat1);
    local lon1 = DecimalDegreesToRadians(lon1);
    local lat2 = DecimalDegreesToRadians(lat2);
    local lon2 = DecimalDegreesToRadians(lon2);
    local d1 = math.acos(math.sin(lat1)*math.sin(lat2)+math.cos(lat1)*math.cos(lat2)*math.cos(lon1-lon2));
    return d1;
end
Mercator.getDistance = getDistance

local getDistanceKM = function( lat1,lon1,lat2,lon2 )
    local d1 = getDistance( lat1,lon1,lat2,lon2 );
    return d1*Mercator.radiusKM;
end
Mercator.getDistanceKM = getDistanceKM

local checkBorders = function()
    local centerX = display.contentCenterX
    local centerY = display.contentCenterY
    if Mercator.projection.x > centerX + Mercator.map.contentWidth/2 then
        Mercator.projection.x = centerX + Mercator.map.contentWidth/2
    elseif Mercator.projection.x < centerX - Mercator.map.contentWidth/2 then
        Mercator.projection.x = centerX - Mercator.map.contentWidth/2
    end
    if Mercator.projection.y > centerY + Mercator.map.contentHeight/2 then
        Mercator.projection.y = centerY + Mercator.map.contentHeight/2
    elseif Mercator.projection.y < centerY - Mercator.map.contentHeight/2 then
        Mercator.projection.y = centerY - Mercator.map.contentHeight/2
    end
end

--------------------------------------------------------------------------------
-- multitouch
--------------------------------------------------------------------------------
function newTrackDot(e)
    local circle = display.newCircle( e.x, e.y, 50 )
    circle.alpha = .5
    circle.xStart = e.x
    circle.yStart = e.y
    circle.ready = true
    circle.go = false
    local rect = e.target
    function circle:touch(e)
        local target = circle
        e.parent = rect
        if (e.phase == "began") then
            display.getCurrentStage():setFocus(target, e.id)
            target.hasFocus = true
            return true
        elseif (target.hasFocus) then
            if (e.phase == "moved") then
                target.x, target.y = e.x, e.y
            else
                display.getCurrentStage():setFocus(target, nil)
                target.hasFocus = false
            end
            rect:touch(e)
            return true
        end
        return false
    end
    circle:addEventListener("touch")
    function circle:tap(e)
        if (e.numTaps == 2) then
            e.parent = rect
            rect:touch(e)
        end
        return true
    end
    if (not isDevice) then
        circle:addEventListener("tap")
    end
    circle:touch(e)
    return circle
end
function dotHasMoved( dot )
    if math.abs(dot.xStart-dot.x)<10 and math.abs(dot.yStart-dot.y)<10 then
        return false
    end
    return true
end
function colorDots(e)
    local p = Mercator.projection
    if #p.dots > 1 then
        for i=1, #p.dots do
            p.dots[i]:setFillColor( 255 )
            p.dots[i].ready = false
        end
        return
    end
    if #p.dots == 1 then
        local dot = p.dots[1]
        if dotHasMoved( dot ) or dot.ready==false then
            dot:setFillColor( 255 )
            dot.ready=false
        else
            timer.performWithDelay(300, function()
                if p.dots[1] ~= nil then
                    if dotHasMoved( p.dots[1] ) == false then
                        p.dots[1]:setFillColor( 0, 255, 0 )
                        p.dots[1].go=true
                    end    
                end
            end )
            --dot:setFillColor( 0, 255, 0 )
            --dot.ready=true
        end
    end
end
function Mercator.projection:touch(e)
    local target = e.target

    if (e.phase == "began") then
        local dot = newTrackDot(e)
        self.dots[ #self.dots+1 ] = dot
        self.prevCentre = calcAvgCentre( self.dots )
        updateTracking( self.prevCentre, self.dots )
        colorDots(e)
        return true
    elseif (e.parent == self) then
        if (e.phase == "moved") then
            local centre, scale = {}, 1
            centre = calcAvgCentre( self.dots )
            updateTracking( self.prevCentre, self.dots )
            if (#self.dots > 1) then
                scale = calcAverageScaling( self.dots )
                if (self.xScale >= 4.0 and scale >=1) or (self.xScale <= .6 and scale <=1) then
                    scale = 1
                end
                self.xScale, self.yScale = self.xScale * scale, self.yScale * scale
            end
            local pt = {}
            pt.x = self.x + (centre.x - self.prevCentre.x)
            pt.y = self.y + (centre.y - self.prevCentre.y)
            checkBorders()
            pt.x = centre.x + ((pt.x - centre.x) * scale)
            pt.y = centre.y + ((pt.y - centre.y) * scale)
            self.x, self.y = pt.x, pt.y
            self.prevCentre = centre
            colorDots(e)
        else
            if (isDevice or e.numTaps == 2) then
                colorDots(e)
                if e.target.ready and e.target.go then
                    local lat,lon
                    lat = getLatFromY( (e.target.y-self.y)/self.yScale )
                    lon = getLonFromX( (e.target.x-self.x)/self.xScale )
                    --[[if system.orientation=="portrait" then
                        lat = getLatFromY( (e.target.y-self.y)/self.yScale )
                        lon = getLonFromX( (e.target.x-self.x)/self.xScale )
                    elseif system.orientation=="landscapeLeft" then
                        lat = getLatFromY( (e.target.x-self.x)/self.yScale )
                        lon = getLonFromX( (-e.target.y+self.y)/self.yScale )
                    elseif system.orientation=="portraitUpsideDown" then
                        lat = getLatFromY( (-e.target.y+self.y)/self.yScale )
                        lon = getLonFromX( (-e.target.x+self.x)/self.yScale )
                    elseif system.orientation=="landscapeRight" then
                        lat = getLatFromY( (-e.target.x+self.x)/self.yScale )
                        lon = getLonFromX( (e.target.y-self.y)/self.yScale )
                    else
                    end]]
                    Mercator.onEvent( { name="tap",phase="ready",target=Mercator,lon=lon,lat=lat } )
                end
                local index = table.indexOf( self.dots, e.target )
                table.remove( self.dots, index )
                e.target:removeSelf()
                self.prevCentre = calcAvgCentre( self.dots )
                updateTracking( self.prevCentre, self.dots )
            end
        end
        return true
    end
    return false
end

--------------------------------------------------------------------------------

local showDistance = function(lat1,lon1,lat2,lon2,distances)
    if Mercator.debug then
        print( "Mercator.showDistance", lat1,lon1,lat2,lon2,distances )
    end
    local distances = distances or {100,500,1000}
    
    local circle = display.newGroup()
    local scale = Mercator.projection.xScale
    local posX1 = getXFromLon( lon1 )
    local posY1 = getYFromLat( lat1 )
    local posX2 = getXFromLon( lon2 )
    local posY2 = getYFromLat( lat2 )
    
    local distancePx = math.sqrt(math.pow(math.abs(posX1-posX2),2)+math.pow(math.abs(posY1-posY2),2))

    local distance = getDistance(lat1, lon1, lat2, lon2)
    local distanceKM = getDistanceKM(lat1, lon1, lat2, lon2)
    
    local circleGreen = display.newCircle( posX1, posY1, distancePx/distanceKM*distances[1] )
    circleGreen:setFillColor(0,255,0,128)
    circle:insert(circleGreen)

    if distanceKM>distances[1] then
        local circleYellow = display.newCircle( posX1, posY1, distancePx/distanceKM*distances[2] )
        circleYellow:setFillColor(255,255,0,64)
        circle:insert(circleYellow)
    end
    
    if distanceKM>distances[2] then
        local circleRed = display.newCircle( posX1, posY1, distancePx/distanceKM*distances[3] )
        circleRed:setFillColor(255,0,0,32)
        circle:insert(circleRed)
    end
    
    local circleGray = display.newCircle( posX1, posY1, distancePx )
    circleGray:setFillColor(128,128,128,32)
    transition.from( circleGray, { time=200, delay=600, xScale=0.1, yScale=0.1 } )
    circle:insert(circleGray)
    Mercator.drawings:insert( circle )
end
Mercator.showDistance = showDistance

local centerDistance = function(lat1,lon1,lat2,lon2,animate)
    if Mercator.debug then
        print( "Mercator.setDistance", lat1,lon1,lat2,lon2,animate )
    end
    animate = animate==true or false
    local scale = Mercator.projection.xScale
    local posX1, posY1, posX2, posY2
    posX1 = getXFromLon(lon1)
    posY1 = getYFromLat(lat1)
    posX2 = getXFromLon(lon2)
    posY2 = getYFromLat(lat2)
    
    local centerX, centerY
    centerX = (posX1+posX2)/2
    centerY = (posY1+posY2)/2
    
    local centerLat = Mercator.getLatFromY(centerY)
    local centerLon = Mercator.getLonFromX(centerX)
    local distance = Mercator.getDistance(lat1, lon1, lat2, lon2)
    local distanceKM = Mercator.getDistanceKM(lat1, lon1, lat2, lon2)
    local distanceScale = Mercator.radiusKM / (distanceKM/math.pi) / 2

    if distanceScale>3 then
        distanceScale = 3
    end
    scale = distanceScale
    
    
--print(    Mercator.getDistanceKM(0,-180,0,170) )

    --[[if distanceScale<1 then
        scale = 1
    elseif distanceScale>3 then
        scale = 3
    else
        scale = distanceScale
    end]]
    Mercator.setCenter(centerLat, centerLon, scale, animate)
end
Mercator.centerDistance = centerDistance

local setCenter = function(lat,lon,scale,animate)
    if Mercator.debug then
        print( "Mercator.setCenter", lat,lon,scale,animate )
    end
    animate = animate==true or false
    scale = scale or Mercator.projection.xScale
    local centerX, centerY, posX, posY
    centerX = display.contentCenterX
    centerY = display.contentCenterY
    posX = centerX - getXFromLon(lon) * scale
    posY = centerY - getYFromLat(lat) * scale
    --[[
    if system.orientation=="portrait" then
        posX = centerX - getXFromLon(lon) * scale
        posY = centerY - getYFromLat(lat) * scale
    elseif system.orientation=="landscapeLeft" then
        posX = centerX - getYFromLat(lat) * scale
        posY = centerY + getXFromLon(lon) * scale
    elseif system.orientation=="portraitUpsideDown" then
        posX = centerX + getXFromLon(lon) * scale
        posY = centerY + getYFromLat(lat) * scale
    elseif system.orientation=="landscapeRight" then
        posX = centerX + getYFromLat(lat) * scale
        posY = centerY - getXFromLon(lon) * scale
    else
    end
    ]]
    if animate ~= false then
        transition.to( Mercator.projection, { time=300, xScale=scale, yScale=scale, x=posX, y=posY
        })
    else
        Mercator.projection.xScale = scale
        Mercator.projection.yScale = scale
        Mercator.projection.x = posX
        Mercator.projection.y = posY
    end
    checkBorders()
    return posX,posY
end
Mercator.setCenter = setCenter

local setFlag = function(color,lat,lon,inverse)
    if Mercator.debug then
        print( "Mercator.setFlag",color,lat,lon,inverse )
    end
    local texture = "lib_dschini_mercator/flag-"..color..".png"
    local flag = display.newGroup()
    flag:setReferencePoint(display.CenterReferencePoint)
    local icon = display.newImageRect(texture, 35, 37 )
    local shadow = display.newImageRect("lib_dschini_mercator/flag-shadow.png", 12, 4 )
    local inverse = inverse or false

    local posX = getXFromLon( lon ) 
    local posY = getYFromLat( lat ) 

    shadow:setReferencePoint( display.BottomCenterReferencePoint )
    shadow.yReference = 0
    shadow.x = posX
    shadow.y = posY
    
    icon:setReferencePoint( display.BottomCenterReferencePoint )
    if inverse then
        icon.xReference = -icon.contentWidth/2+2
        icon.xScale = -0.75--/projection.xScale
        icon.yScale = 0.75--/projection.xScale
        icon.rotation = -45
        icon.x = posX-10
        icon.y = posY-40
        shadow.x = posX-3
        shadow.y = posY+3
        transition.to( icon, { time=400, transition=easing.linear, x=posX, y=posY, rotation=0} )
        transition.to( shadow, { time=200, transition=easing.linear, x=posX, y=posY} )        
    else
        icon.xReference = -icon.contentWidth/2+2
        icon.xScale = 0.75--/projection.xScale
        icon.yScale = 0.75--/projection.xScale
        icon.rotation = 45
        icon.x = posX+10
        icon.y = posY-40
        shadow.x = posX+3
        shadow.y = posY+3
        transition.to( icon, { time=400, transition=easing.linear, x=posX, y=posY, rotation=0} )
        transition.to( shadow, { time=200, transition=easing.linear, x=posX, y=posY} )        
    end
    flag:insert(shadow)
    flag:insert(icon)
    Mercator.controls:insert( flag )
    return flag
end
Mercator.setFlag = setFlag

local disable = function()
    Mercator.projection:removeEventListener( "touch", Mercator.projection )
end
Mercator.disable = disable

local enable = function()
    Mercator.projection:addEventListener( "touch", Mercator.projection )
end
Mercator.enable = enable

local removeControls = function()
    if Mercator.controls.numChildren > 0 then
        for i=Mercator.controls.numChildren,1,-1 do
            Mercator.controls[i]:removeSelf()
            Mercator.controls[i] = nil
        end
    end 
end
Mercator.removeControls = removeControls

local removeDrawings = function()
    if Mercator.drawings.numChildren > 0 then
        for i=Mercator.drawings.numChildren,1,-1 do
            Mercator.drawings[i]:removeSelf()
            Mercator.drawings[i] = nil
        end
    end 
end
Mercator.removeDrawings = removeDrawings

local setSeaColor = function(r,g,b)
    Mercator.background:setFillColor(r,g,b)
end
Mercator.setSeaColor = setSeaColor

local setMapColor = function(r,g,b)
    Mercator.map:setFillColor(r,g,b)
end
Mercator.setMapColor = setMapColor

local setBordersColor = function(r,g,b)
    Mercator.borders:setFillColor(r,g,b)
end
Mercator.setBordersColor = setBordersColor

local function onSystemEvent( event )
    if event.type == "applicationExit" then
        Runtime:removeEventListener( "system", onSystemEvent )
        if Mercator.db and Mercator.db:isopen() then
            Mercator.db:close()
        end
    end
end

function Mercator:new(owner,debug)
    Mercator.owner = owner or display.newGroup()
    Mercator.debug = debug or false
    
    --[[
    Mercator.map = display.newImageRect( "lib_dschini_mercator/mercator.png", 960, 960 )
    Mercator.background = display.newRect(
        0,0,
        Mercator.map.contentWidth+display.contentWidth,
        Mercator.map.contentHeight+display.contentWidth -- Wichtig hier nicht height da nach rotation sonst ein schwarzer balken oben ist
    )
    Mercator.background:setFillColor(255, 255, 255)
    ]]
    
    -- http://www.base2solutions.com/walkabout/Corona%20Tips.html

    local scale = tonumber(string.format("%." .. (3) .. "f", 1/display.contentScaleY)) --1.067
    if scale == 0.5 then ImageSuffix = "-960"
    elseif scale == 1.067 then ImageSuffix = "-2048"
    elseif scale == 2.133 then ImageSuffix = "-4096"
    else ImageSuffix = ""  -- 1 -- iphone4, iphone5
    end

    Mercator.background = display.newRect(
        0,0,
        960+display.contentWidth,
        960+display.contentWidth -- Wichtig hier nicht height da nach rotation sonst ein schwarzer balken oben ist
    )
    Mercator.background:setFillColor(0, 0, 255)
    
    Mercator.map = display.newRect(
        0,0,
        960+display.contentWidth,
        960+display.contentWidth -- Wichtig hier nicht height da nach rotation sonst ein schwarzer balken oben ist
    )
    Mercator.map:setFillColor(200, 200, 200)
    local mask = graphics.newMask( "lib_dschini_mercator/mercator-land"..ImageSuffix..".png" )
    Mercator.map:setMask( mask )
    Mercator.map.xScale = .5
    Mercator.map.yScale = .5
    
    Mercator.borders = display.newRect(
        0,0,
        960+display.contentWidth,
        960+display.contentWidth -- Wichtig hier nicht height da nach rotation sonst ein schwarzer balken oben ist
    )
    Mercator.borders:setFillColor(0, 200, 200)
    local mask = graphics.newMask( "lib_dschini_mercator/mercator-borders"..ImageSuffix..".png" )
    Mercator.borders:setMask( mask )
    Mercator.borders.xScale = .5
    Mercator.borders.yScale = .5
    
    Mercator.projection:insert( Mercator.background )
    Mercator.projection:insert( Mercator.map )
    Mercator.projection:insert( Mercator.borders )
    Mercator.projection:insert( Mercator.drawings )
    Mercator.projection:insert( Mercator.controls )
    Mercator.owner:insert( Mercator.projection )

    Mercator.projection.xScale = 1
    Mercator.projection.yScale = 1
    Mercator.controls.x = display.contentCenterX
    Mercator.controls.y = display.contentCenterY
    Mercator.drawings.x = display.contentCenterX
    Mercator.drawings.y = display.contentCenterY
    Mercator.background.x = display.contentCenterX
    Mercator.background.y = display.contentCenterY
    Mercator.map.x = display.contentCenterX
    Mercator.map.y = display.contentCenterY
    Mercator.borders.x = display.contentCenterX
    Mercator.borders.y = display.contentCenterY
    Mercator.projection:setReferencePoint( display.CenterReferencePoint )
    
    Mercator.db = sqlite3.open( Mercator.dbPath )

    if Mercator.debug then
        local cross = display.newGroup()
        local crossH = display.newLine( 0,display.contentCenterY, display.contentWidth,display.contentCenterY )
        local crossMercator = display.newLine( display.contentCenterX,0, display.contentCenterX,display.contentHeight )
        cross:insert( crossMercator )
        cross:insert( crossH )
        Mercator.owner:insert( cross )    
        crossH:setColor( 0, 0, 255, 255 )
        crossH.width = 1 
        crossMercator:setColor( 0, 0, 255, 255 )
        crossMercator.width = 1 
    end
    
    --Runtime:addEventListener( "orientation", onOrientationChange )
    Runtime:addEventListener( "system", onSystemEvent )

end

return Mercator
