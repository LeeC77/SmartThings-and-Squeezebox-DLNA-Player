/**
 *  MediaRenderer Service Manager v 2.1.LC
 *
 *  Author: SmartThings - Ulises Mujica 
 */
 
 // LC useful resourse: http://docs.smartthings.com/en/latest/cloud-and-lan-connected-device-types-developers-guide/building-lan-connected-device-types/building-the-service-manager.html?highlight=parsedEvent
 //						http://upnp.org/specs/arch/UPnP-arch-DeviceArchitecture-v1.1.pdf
 
 
definition(
	name: "LC MediaRenderer (Connect)",
	namespace: "LeeC77",
	author: "SmartThings - Ulises Mujica",
	description: "Allows you to control your Media Renderer from the SmartThings app. Perform basic functions like play, pause, stop, change track, and check artist and song name from the Things screen.",
	category: "SmartThings Labs",
	singleInstance: true,
	iconUrl: "https://graph.api.smartthings.com/api/devices/icons/st.secondary.smartapps-tile?displaySize=2x",
    iconX2Url: "https://graph.api.smartthings.com/api/devices/icons/st.secondary.smartapps-tile?displaySize=2x"
)

preferences {
	
    page(name: "MainPage", title: "Search and config your Media Renderers", install:true, uninstall: true){
    	section("") {
            href(name: "discover",title: "Discovery process",required: false,page: "mediaRendererDiscovery",description: "tap to start searching")
        }
        
        /////////////////////////////////////////// LC ///////////////////////////////////////////////////
        section("SBS Response"){
        	input "Director1", "capability.Sensor", title: "Which DLNA Subscriber reports connection status?", multiple: false, required: false  //LC
        }
        /////////////////////////////////////////// LC ///////////////////////////////////////////////////
        
        section("Options", hideable: true, hidden: true) {
            input("refreshMRInterval", "number", title:"Enter ip changes interval (min)",defaultValue:"15", required:false)
            input("updateMRInterval", "number", title:"Enter refresh players interval (min)",defaultValue:"3", required:false)
        }
        section("") {
            href(name: "watchDog",title: "WatchDog",required: false,page: "watchDogPage",description: "tap to config WatchDog")
        }
    }
    page(name: "mediaRendererDiscovery", title:"Discovery Started!")
    page(name: "watchDogPage", title:"WatchDog")
}

def mediaRendererDiscovery()// Loops while on Discovery page
{
    // log.trace "mediaRendererDiscovery() state.subscribe ${state.subscribe}"  // LC add this back
    if(canInstallLabs())
	{

        int mediaRendererRefreshCount = !state.mediaRendererRefreshCount ? 0 : state.mediaRendererRefreshCount as int
        // set the local mediaRendererRefreshCount = 0 if a count doesn't exist else to the known count. 
		state.mediaRendererRefreshCount = mediaRendererRefreshCount + 1
        // set the known mediaRendererRefreshCount to the local +1 (used for timing of discovery calls)
        
		def refreshInterval = 5

		def options = mediaRenderersDiscovered() ?: [] //LC set options to mediaRenderersDiscovered if exists else an empty list

		def numFound = options.size() ?: 0 // set the number found to the size of options if it exists

		if(!state.subscribe) {
			subscribe(location, null, locationHandler, [filterEvents:false]) // subscribes location events (the SSDP search response)) using the locationHandler
			state.subscribe = true
		}
		// mediaRenderer discovery request every 5 //25 seconds
		if((mediaRendererRefreshCount % 8) == 0) {
			discoverMediaRenderers()
		}
		// setup.xml request every 3 seconds except on discoveries
		if(((mediaRendererRefreshCount % 1) == 0) && ((mediaRendererRefreshCount % 8) != 0)) {
			verifyMediaRendererPlayer()
		}
		return dynamicPage(name:"mediaRendererDiscovery", title:"Discovery Started!", nextPage:"", refreshInterval:refreshInterval) {
			section("Please wait while we discover your MediaRenderer. Discovery can take five minutes or more, so sit back and relax! Select your device below once discovered.") {
				input "selectedMediaRenderer", "enum", required:false, title:"Select Media Renderer (${numFound} found)", multiple:true, options:options
			} // LC Select the found media renderers using a dynamic page
		}
	}
	else
	{
		def upgradeNeeded = """To use SmartThings Labs, your Hub should be completely up to date.
		To update your Hub, access Location Settings in the Main Menu (tap the gear next to your location name), select your Hub, and choose "Update Hub"."""

		return dynamicPage(name:"mediaRendererDiscovery", title:"Upgrade needed!", nextPage:"", install:false, uninstall: true) {
			section("Upgrade") {
				paragraph "$upgradeNeeded"
			}
		}
	}
}

private discoverMediaRenderers()// sends discovery request to ST Hub
{
	sendHubCommand(new physicalgraph.device.HubAction("lan discovery urn:schemas-upnp-org:device:MediaRenderer:1", physicalgraph.device.Protocol.LAN))
}

// verifies the discovered renderers
private verifyMediaRendererPlayer() {
	def devices = getMediaRendererPlayer().findAll { it?.value?.verified != true } // creates a list of devices that are not verified
	devices.each {
		// verifyMediaRenderer((it?.value?.ip + ":" + it?.value?.port), it?.value?.ssdpPath) // for each unverified device get renderer info - LC this need to reapplied 
        String tempString = it?.value?.ssdpUSN  // LC get uuid
        def splitString = tempString.split(":") // LC
        log.trace "LC trace verifying ${splitString[1]}"
        verifyMediaRenderer((it?.value?.ip + ":" + it?.value?.port), it?.value?.ssdpPath, splitString[1])//LC passes UuId also. For each unverified device get renderer info 
	}
}
// requests the UPnP device description (.xml)
private verifyMediaRenderer(String deviceNetworkId, String ssdpPath, String deviceUuId) { // LC , String deviceUuId new
	String ip = getHostAddress(deviceNetworkId)
    	if(!ssdpPath){
		ssdpPath = "/"
	}
    if (deviceNetworkId){ // LC allow all
		//sendHubCommand(new physicalgraph.device.HubAction("""GET $ssdpPath HTTP/1.1\r\nHOST: $ip\r\n\r\n""", physicalgraph.device.Protocol.LAN, "${deviceNetworkId}")) //original
        sendHubCommand(new physicalgraph.device.HubAction("""GET $ssdpPath HTTP/1.1\r\nHOST: $ip\r\n\r\n""", physicalgraph.device.Protocol.LAN, "${deviceUuId}", [callback: locationHandler]))         
	}
}

Map mediaRenderersDiscovered() { // defines the list of players that are discovered as a map for selection.
	def vmediaRenderers = getVerifiedMediaRendererPlayer()
   	def map = [:]
	vmediaRenderers.each {
    	def value = "${it.value.name}"
		def key
		// def key = it.value.ip + ":" + it.value.port // LC original line
		        if ((it.value.model) =~/Squeezebox/){   //LC
                    key = it.value.UuId                        //LC
                    }
                else {
                	key = it.value.ip + ":" + it.value.port
                    }
        log.trace "LC Trace added to selection list ${key}" //LC
		map["${key}"] = value
	}
    //log.trace "LC Trace Listed: ${map}" //LC
	map
}

def getMediaRendererPlayer()
{
	state.mediaRenderers = state.mediaRenderers ?: [:] // known renderers = known renders if they exist else empty list
    //log.trace "LC 1: ($state.mediaRenderers)"   //LC
}

def getVerifiedMediaRendererPlayer() // gets list of known verified players
{
	getMediaRendererPlayer().findAll{ it?.value?.verified == true }
    
}

def installed() {
	log.trace "installed()"
	//initialize()
}

def updated() {
	log.trace "updated()"
	if (selectedMediaRenderer) addMediaRenderer()
	unsubscribe()
	state.subscribe = false
    state.child = "Empty"   //LC
    unschedule()
    clearMediaRenderers()
    scheduleTimer()
    timerAll()
    scheduleActions()
    refreshAll()
    syncDevices()
    subscribeToEvents()
    subscribe (Director1,"dlnaresponse", subscribeeventHandler) //LC
}

def uninstalled() {
	def devices = getChildDevices()
	devices.each {
		deleteChildDevice(it.deviceNetworkId)
	}
}

def initialize() {
	// remove location subscription aftwards
    log.trace "initialize()"
    subscribe (Director1,"sbsResponse", subscribeeventHandler)//
    subscribe (Director2,"sbsResponse", subscribeeventHandler)//
    scheduledRefreshHandler()
}

///////////////////////////////////////LC ///////////////////////////////////
def subscribeeventHandler(response){

    /* Need to do: 
     More economically send the Rest statements from the DLNA Player
     Create a LAN Handler automatically
    */
	//log.trace "LC Trace Response recieved ${response.value}"
    //Is there a cookie for squeezebox this looks like " set-cookie:Squeezebox-player="
    def result=response.value
    def String cookie ="NoCOOKIE"
    def String scookie ="set-cookie"
    def String cookieName = "Squeezebox-player="
    def devices
    state.child=getStateValue('child')
    result = parseLanMessage(result)  // need this
    
    // Find the Cookie if there is one.
    //log.trace " LC Trace Result = ${result}"   // LC
    def header = result.headers // The cookie is in the header map
    header.each {key, val ->    // LC  could be more efficent here 
    	//log.debug "result key: $key, value: $val"
    	if (key =~ scookie) {
        	if (val =~ cookieName) {
        		cookie = val-cookieName // Remove the cookie name
        		cookie=cookie.split(';')[0] // Remove everythig following
            }
        }
    }
    //log.trace "LC Trace Squeezebox cookie is ${cookie}" //LC
	//Get all child devices if we have a cookie
    if (cookie != "NoCOOKIE"){
    	devices = getChildDevices()
    	devices.each { device ->
        	if ("${device.getDataValue('hostaddress')}" =~ cookie) {
               	//state.child = device.getDataValue('dni') // Store in a state variable as not all messages have a SetCookie
                putStateValue ("child", "${device.getDataValue('dni')}")
                log.trace " LC Trace 1 cookie, ${cookie} is for ${device.label}, ${state.child}"
            }
    	}
    }
    
    //Send Events
    if ((state.child != "Empty") & (cookie == "NoCOOKIE")){ // If we have a child device and its a reply without a cookie
   			
    	
    def d = getChildDevice(state.child) // Get the child device with the stored dni
        
            log.trace " LC Trace Sending to --> ${d.label}"
     if (d){
            d.subscribeResp(response) // This line gives an error but sends anyway
            log.trace " LC Trace Sent to --> ${d.label}"
            //def evt = sendEvent(d, name: "subscribeResp", value : response)
            //return evt
        } else {
        	log.trace "Unknown child device"
        }
    } else {
    	log.trace "No need to forward the message conatiing Set-cookie"
   }
   log.trace "exit"
}
 /////////////////////////////SmartThings work round //////////////////////
def getStateValue(key) {
    return state[key]
}

def putStateValue(key, value){
	log.trace " $key, $value" 
	state[key]=value
	}


///////////////////////////////////////LC ///////////////////////////////////

def clearMediaRenderers(){ // LC resets the list of all the renderer devices in the state variable.
	log.trace "clearMediaRenderers()"
	def devices = getChildDevices()
    def player
    def players = [:]
    devices.each { device ->
        player = getMediaRendererPlayer().find{ it?.value?.uuid == device.getDataValue('uuid') }
        if (player){
        	players << player
        }
	}
    state.mediaRenderers = players
    
}

def scheduledRefreshHandler(){

}

def scheduledTimerHandler() {
    timerAll()
}

def scheduledActionsHandler() {
    syncDevices()
	//runIn(60, scheduledRefreshHandler) 
}

private scheduleTimer() {
    def cron = "0 0/1 * * * ?"
   	schedule(cron, scheduledTimerHandler)
}

private scheduleActions() {
    def minutes = Math.max(settings.refreshMRInterval.toInteger(),1)
    def cron = "0 0/${minutes} * * * ?"
   	schedule(cron, scheduledActionsHandler)
}

private syncDevices() {
	if(!state.subscribe) {
		subscribe(location, null, locationHandler, [filterEvents:false])
		state.subscribe = true
	}
	discoverMediaRenderers()
}

private timerAll(){
    state.actionTime = new Date().time
	childDevices*.poll()
}

private refreshAll(){
	childDevices*.refresh()
}

def addMediaRenderer() {
	def players = getVerifiedMediaRendererPlayer()
    // log.trace "LC trace verified players look like ==> ${players}" //LC
	def runSubscribe = false
	selectedMediaRenderer.each { dni -> // for  each selected media renderer see if there ia already a device if not add one.
    //log.trace " LC Trace selected renderers are ${selectedMediaRenderer}" // LC
		def d = getChildDevice(dni)
        log.trace " LC Trace Check for an existing device with ID ${dni}, result= ${d}" // LC
		if(!d) {
        
         
			//def newPlayer = players.find { (it.value.ip + ":" + it.value.port) == dni } // LC original line, selects the entry in a copy of the verified list that has the same ip:port identity selected.
            //def newPlayer = players.find { (it.value.UuId)  == dni } // LC
			def newPlayer
            def hostaddress
            if (players.find { (it.value.ip + ":" + it.value.port) == dni }){ //LC
            	newPlayer = players.find { (it.value.ip + ":" + it.value.port) == dni }
            	log.trace "LC Trace got a none Squeezebox" //LC
            	} //LC
            if (players.find { (it.value.UuId)  == dni }){ //LC
            	newPlayer = players.find { (it.value.UuId)  == dni }
                log.trace "LC Trace got a Squeezebox" //LC
            	} //LC
            
            //def value = "${it.value.name}" // LC delete me
            
            // If its a Squeeze box then include ssdpPath as part of host aadress
            def check= "${newPlayer.value.model}"
            //log.trace "LC Trace ${check}"
            if ((newPlayer.value.model) =~/Squeezebox/){  // LC
            hostaddress = "${newPlayer.value.ip}:${newPlayer.value.port}:${newPlayer.value.ssdpPath}" //LC 
            newPlayer = players.find { (it.value.UuId)  == dni } //LC
            log.trace "LC Trace ${newPlayer.value.model} found"  // LC
            } // LC
            else {
            hostaddress = "${newPlayer.value.ip}:${newPlayer.value.port}" //LC
            newPlayer = players.find { (it.value.ip + ":" + it.value.port) == dni } //LC
            }         
            log.trace "LC Trace creating new device ${dni} host = ${hostaddress}" //LC
            if (newPlayer){
				//d = addChildDevice("mujica", "DLNA Player", dni, newPlayer?.value.hub, [label:"${newPlayer?.value.name} Speaker","data":["model":newPlayer?.value.model,"avtcurl":newPlayer?.value.avtcurl,"avteurl":newPlayer?.value.avteurl,"rccurl":newPlayer?.value.rccurl,"rceurl":newPlayer?.value.rceurl,"pcurl":newPlayer?.value.pcurl,"peurl":newPlayer?.value.peurl,"udn":newPlayer?.value.udn,"dni":dni]])   // LC Original
                d = addChildDevice("LeeC77", "LC DLNA Player", dni, newPlayer?.value.hub, [label:"${newPlayer?.value.name} Speaker","data":["model":newPlayer?.value.model,"hostaddress":hostaddress,"avtcurl":newPlayer?.value.avtcurl,"avteurl":newPlayer?.value.avteurl,"rccurl":newPlayer?.value.rccurl,"rceurl":newPlayer?.value.rceurl,"pcurl":newPlayer?.value.pcurl,"peurl":newPlayer?.value.peurl,"udn":newPlayer?.value.udn,"dni":dni]])  // LC restore
				log.trace "Trace LC Device created dni ${dni} " // LC
            }
			runSubscribe = true
		} 
	}
}

def locationHandler(evt) { //Handles the Hub events returned for each Renderer
	// log.trace " LC in locationHandler"
	def description = evt.description
	def hub = evt?.hubId
	def parsedEvent = parseEventMessage(description)
	def msg = parseLanMessage(description)

    //log.trace " LC in locationHandler post Parse (Description = $description)"
    if (parsedEvent.headers){
    	def header = new String(parsedEvent?.headers?.decodeBase64())
		//log.trace "Header ==> $header" // LC Can add back in
    }
    if (parsedEvent.body){
    	def body = new String(parsedEvent.body.decodeBase64())
        // log.trace "Body ==> $body"// LC Can add back in
    }      
	if (msg?.headers?.sid)
	{
		childDevices*.each { childDevice ->
		    if(childDevice.getDataValue('subscriptionId') == ((msg?.headers?.sid ?:"") - "uuid:")|| childDevice.getDataValue('subscriptionId1') == ((msg?.headers?.sid ?:"") - "uuid:")){
		       childDevice.parse(description)
		    }
		}
	}
    
	parsedEvent << ["hub":hub]
    
	if (parsedEvent?.ssdpTerm?.contains("urn:schemas-upnp-org:device:MediaRenderer:1"))
	{ //SSDP DISCOVERY EVENTS adds new Renderess to list and maintains list parameters if they change.
    	// log.trace "LC trace Found a renderer"
		def mediaRenderers = getMediaRendererPlayer() // get the know list of Renderers
		if (!(mediaRenderers."${parsedEvent.ssdpUSN.toString()}"))
		{ //mediaRenderer does not exist so add it to the list

			mediaRenderers << ["${parsedEvent.ssdpUSN.toString()}":parsedEvent]
            log.trace "LC trace Found a new renderer  ==> ${parsedEvent.ssdpUSN.toString()}"
		}
		else
		{ // update the values
			
			def d = mediaRenderers."${parsedEvent.ssdpUSN.toString()}" // LC Original  
            boolean deviceChangedValues = false
            if(d.ip != parsedEvent.ip || d.port != parsedEvent.port) {
            	log.trace "LC trace updating values of Renderer"
				d.ip = parsedEvent.ip
				d.port = parsedEvent.port
				deviceChangedValues = true
			}
			if (deviceChangedValues) {
                def children = getChildDevices()
				children.each {
                    if (parsedEvent.ssdpUSN.toString().contains(it.getDataValue("udn"))) {
						it.setDeviceNetworkId((parsedEvent.ip + ":" + parsedEvent.port)) //could error if device with same dni already exists
						it.updateDataValue("dni", (parsedEvent.ip + ":" + parsedEvent.port))
                        it.refresh()
						log.trace "Updated Device IP"

					}
				}
			}
		}
	}
	else if (parsedEvent.headers && parsedEvent.body)
	{ // MEDIARENDER RESPONSES
    	def headerString = new String(parsedEvent?.headers?.decodeBase64())
		def bodyString = new String(parsedEvent.body.decodeBase64())
        //log.trace "LC trace  MEDIARENDER RESPONSES --> ($bodyString,## $headerString)<<<<<"    //LC
		def type = (headerString =~ /Content-Type:.*/) ? (headerString =~ /Content-Type:.*/)[0] : null
		def body
        def device
		if (bodyString?.contains("xml"))
		{ // description.xml response (application/xml)
			body = new XmlSlurper().parseText(bodyString)
			log.trace "MEDIARENDER RESPONSES ${body?.device?.modelName?.text()}" // LC Restore this line
			// Avoid add sonos devices	
            device = body?.device
            //log.trace "LC trace device --> ($device)"
            body?.device?.deviceList?.device?.each{
                if (it?.deviceType?.text().contains("urn:schemas-upnp-org:device:MediaRenderer:1")) {
                    device = it
                }
            }
            
			if ( device?.deviceType?.text().contains("urn:schemas-upnp-org:device:MediaRenderer:1"))
			{
				def avtcurl = ""
				def avteurl = ""
				def rccurl = ""
				def rceurl = ""
                def pcurl = ""
				def peurl = ""    
                
				device?.serviceList?.service?.each{
				  if (it?.serviceType?.text().contains("AVTransport")) {
						avtcurl = it?.controlURL?.text().startsWith("/")? it?.controlURL.text() : "/" + it?.controlURL.text()
						avteurl = it?.eventSubURL?.text().startsWith("/")? it?.eventSubURL.text() : "/" + it?.eventSubURL.text()
					}
					if (it?.serviceType?.text().contains("RenderingControl")) {
						rccurl = it?.controlURL?.text().startsWith("/")? it?.controlURL?.text() : "/" + it?.controlURL?.text()
						rceurl = it?.eventSubURL?.text().startsWith("/")? it?.eventSubURL?.text() : "/" + it?.eventSubURL?.text()
					}
                    if (it?.serviceType?.text().contains("Party")) {
						pcurl = it?.controlURL?.text().startsWith("/")? it?.controlURL?.text() : "/" + it?.controlURL?.text()
						peurl = it?.eventSubURL?.text().startsWith("/")? it?.eventSubURL?.text() : "/" + it?.eventSubURL?.text()
					}
				}
				def mediaRenderers = getMediaRendererPlayer() // get the know list of Renderers
				def player = mediaRenderers.find {it?.key?.contains(device?.UDN?.text())} // LC puts into player, if we have already in mediaRenderers a device with the UDN text responded.
                /////////////////////////////////////////////////////////LC
                String tempString = player?.value?.ssdpUSN  // LC get uuid
        		def splitString = tempString.split(":") // LC
                def uuid = splitString[1]
        		//log.trace "LC Trace Getting UuId of player to apend. ${uuid}"
                ////////////////////////////////////////////////////////LC

				if (player)
				{
                	
					//player.value << [name:device?.friendlyName?.text(),model:device?.modelName?.text(), serialNumber:device?.UDN?.text(),verified: true,avtcurl:avtcurl,avteurl:avteurl,rccurl:rccurl,rceurl:rceurl,pcurl:pcurl,peurl:peurl,udn:device?.UDN?.text()] // LC original line
					player.value << [name:device?.friendlyName?.text(),model:device?.modelName?.text(), serialNumber:device?.UDN?.text(), UuId: uuid, verified: true,avtcurl:avtcurl,avteurl:avteurl,rccurl:rccurl,rceurl:rceurl,pcurl:pcurl,peurl:peurl,udn:device?.UDN?.text()]
                    log.trace "LC trace verified ${device?.UDN?.text()}, ${name:device?.friendlyName?.text()}"
                    //tempString = player?.value?.UuId //LC can delete only used for trace
                    //log.trace "LC trace verified ${player.value}" //LC
                    //log.trace "LC trace verified ${tempString}" //LC
                }
			}
		}
		else if(type?.contains("json"))
		{ //(application/json)
			body = new groovy.json.JsonSlurper().parseText(bodyString)
		}
        //LC
        else {// check for anything fialing
        	log.trace " LC Trace  Everything failed "
        }
	}
}

private def parseEventMessage(Map event) {
	//handles mediaRenderer attribute events
	return event
}

private def parseEventMessage(String description) {
	//log.trace "LC Trace  Parsing "
	def event = [:]
	def parts = description.split(',')
	parts.each { part ->
		part = part.trim()
        //log.trace "LC Trace ${part}"   // LC
		if (part.startsWith('devicetype:')) {
			def valueString = part.split(":")[1].trim()
			event.devicetype = valueString
		}
		else if (part.startsWith('mac:')) {
			def valueString = part.split(":")[1].trim()
			if (valueString) {
				event.mac = valueString
			}
		}
		else if (part.startsWith('networkAddress:')) {
			def valueString = part.split(":")[1].trim()
			if (valueString) {
				event.ip = valueString
			}
		}
		else if (part.startsWith('deviceAddress:')) {
			def valueString = part.split(":")[1].trim()
			if (valueString) {
				event.port = valueString
			}
		}
		else if (part.startsWith('ssdpPath:')) {
			def valueString = part.split(":")[1].trim()
			if (valueString) {
				event.ssdpPath = valueString
			}
		}
		else if (part.startsWith('ssdpUSN:')) {
			part -= "ssdpUSN:"
			def valueString = part.trim()
			if (valueString) {
				event.ssdpUSN = valueString
			}
		}
		else if (part.startsWith('ssdpTerm:')) {
			part -= "ssdpTerm:"
			def valueString = part.trim()
			if (valueString) {
				event.ssdpTerm = valueString
			}
		}
		else if (part.startsWith('headers')) {
			part -= "headers:"
			def valueString = part.trim()
			if (valueString) {
				event.headers = valueString
			}
		}
		else if (part.startsWith('body')) {
			part -= "body:"
			def valueString = part.trim()
			if (valueString) {
				event.body = valueString
			}
		}
	}

	if (event.devicetype == "04" && event.ssdpPath =~ /[a-fA-F0-9]{8}-[a-fA-F0-9]{4}-[a-fA-F0-9]{4}-[a-fA-F0-9]{4}-[a-fA-F0-9]{12}/ && !event.ssdpUSN && !event.ssdpTerm){
        def matcher = event.ssdpPath =~ /[a-fA-F0-9]{8}-[a-fA-F0-9]{4}-[a-fA-F0-9]{4}-[a-fA-F0-9]{4}-[a-fA-F0-9]{12}/ 
        def ssdpUSN = matcher[0]
        event.ssdpUSN = "uuid:$ssdpUSN::urn:schemas-upnp-org:device:MediaRenderer:1"
        event.ssdpTerm = "urn:schemas-upnp-org:device:MediaRenderer:1"
    }
	event
}


/////////CHILD DEVICE METHODS
def parse(childDevice, description) {
	def parsedEvent = parseEventMessage(description)

	if (parsedEvent.headers && parsedEvent.body) {
		def headerString = new String(parsedEvent.headers.decodeBase64())
		def bodyString = new String(parsedEvent.body.decodeBase64())

		def body = new groovy.json.JsonSlurper().parseText(bodyString)
	} else {
		return []
	}
}

private Integer convertHexToInt(hex) {
	Integer.parseInt(hex,16)
}

private String convertHexToIP(hex) {
	[convertHexToInt(hex[0..1]),convertHexToInt(hex[2..3]),convertHexToInt(hex[4..5]),convertHexToInt(hex[6..7])].join(".")
}

private getHostAddress(d) {
	def parts = d.split(":")
	def ip = convertHexToIP(parts[0])
	def port = convertHexToInt(parts[1])
	return ip + ":" + port
}

private Boolean canInstallLabs()
{
	return hasAllHubsOver("000.011.00603")
}

private Boolean hasAllHubsOver(String desiredFirmware)
{
	return realHubFirmwareVersions.every { fw -> fw >= desiredFirmware }
}

private List getRealHubFirmwareVersions()
{
	return location.hubs*.firmwareVersionString.findAll { it }
}



/*watch dog*/


def watchDogPage() {
	dynamicPage(name: "watchDogPage") {
		def anythingSet = anythingSet()

        if (anythingSet) {
			section("Verify Timer When"){
				ifSet "motion", "capability.motionSensor", title: "Motion Here", required: false, multiple: true
				ifSet "contact", "capability.contactSensor", title: "Contact Opens", required: false, multiple: true
				ifSet "contactClosed", "capability.contactSensor", title: "Contact Closes", required: false, multiple: true
				ifSet "acceleration", "capability.accelerationSensor", title: "Acceleration Detected", required: false, multiple: true
				ifSet "mySwitch", "capability.switch", title: "Switch Turned On", required: false, multiple: true
				ifSet "mySwitchOff", "capability.switch", title: "Switch Turned Off", required: false, multiple: true
				ifSet "arrivalPresence", "capability.presenceSensor", title: "Arrival Of", required: false, multiple: true
				ifSet "departurePresence", "capability.presenceSensor", title: "Departure Of", required: false, multiple: true
				ifSet "smoke", "capability.smokeDetector", title: "Smoke Detected", required: false, multiple: true
				ifSet "water", "capability.waterSensor", title: "Water Sensor Wet", required: false, multiple: true
                ifSet "temperature", "capability.temperatureMeasurement", title: "Temperature", required: false, multiple: true
                ifSet "powerMeter", "capability.powerMeter", title: "Power Meter", required: false, multiple: true
                ifSet "energyMeter", "capability.energyMeter", title: "Energy", required: false, multiple: true
                ifSet "signalStrength", "capability.signalStrength", title: "Signal Strength", required: false, multiple: true
				ifSet "button1", "capability.button", title: "Button Press", required:false, multiple:true //remove from production
				ifSet "triggerModes", "mode", title: "System Changes Mode", required: false, multiple: true
			}
		}
		def hideable = anythingSet || app.installationState == "COMPLETE"
		def sectionTitle = anythingSet ? "Select additional triggers" : "Verify Timer When..."

		section(sectionTitle, hideable: hideable, hidden: true){
			ifUnset "motion", "capability.motionSensor", title: "Motion Here", required: false, multiple: true
			ifUnset "contact", "capability.contactSensor", title: "Contact Opens", required: false, multiple: true
			ifUnset "contactClosed", "capability.contactSensor", title: "Contact Closes", required: false, multiple: true
			ifUnset "acceleration", "capability.accelerationSensor", title: "Acceleration Detected", required: false, multiple: true
			ifUnset "mySwitch", "capability.switch", title: "Switch Turned On", required: false, multiple: true
			ifUnset "mySwitchOff", "capability.switch", title: "Switch Turned Off", required: false, multiple: true
			ifUnset "arrivalPresence", "capability.presenceSensor", title: "Arrival Of", required: false, multiple: true
			ifUnset "departurePresence", "capability.presenceSensor", title: "Departure Of", required: false, multiple: true
			ifUnset "smoke", "capability.smokeDetector", title: "Smoke Detected", required: false, multiple: true
			ifUnset "water", "capability.waterSensor", title: "Water Sensor Wet", required: false, multiple: true
			ifUnset "temperature", "capability.temperatureMeasurement", title: "Temperature", required: false, multiple: true
            ifUnset "signalStrength", "capability.signalStrength", title: "Signal Strength", required: false, multiple: true
            ifUnset "powerMeter", "capability.powerMeter", title: "Power Meter", required: false, multiple: true
            ifUnset "energyMeter", "capability.energyMeter", title: "Energy Meter", required: false, multiple: true
			ifUnset "button1", "capability.button", title: "Button Press", required:false, multiple:true //remove from production
			ifUnset "triggerModes", "mode", title: "System Changes Mode", description: "Select mode(s)", required: false, multiple: true
		}
	}
}

private anythingSet() {
	for (name in ["motion","contact","contactClosed","acceleration","mySwitch","mySwitchOff","arrivalPresence","departurePresence","smoke","water", "temperature","signalStrength","powerMeter","energyMeter","button1","timeOfDay","triggerModes","timeOfDay"]) {
		if (settings[name]) {
			return true
		}
	}
	return false
}

private ifUnset(Map options, String name, String capability) {
	if (!settings[name]) {
		input(options, name, capability)
	}
}

private ifSet(Map options, String name, String capability) {
	if (settings[name]) {
		input(options, name, capability)
	}
}

private takeAction(evt) {
	def eventTime = new Date().time
	if (eventTime > ( 60000 + 3 * 1000 * 60 + state.actionTime?:0)) {
		scheduleTimer()
        timerAll()
	}
}

def eventHandler(evt) {
    takeAction(evt)
}

def modeChangeHandler(evt) {
	if (evt.value in triggerModes) {
		eventHandler(evt)
	}
}

def subscribeToEvents() {
	//subscribe(app, appTouchHandler)
	subscribe(contact, "contact.open", eventHandler)
	subscribe(contactClosed, "contact.closed", eventHandler)
	subscribe(acceleration, "acceleration.active", eventHandler)
	subscribe(motion, "motion.active", eventHandler)
	subscribe(mySwitch, "switch.on", eventHandler)
	subscribe(mySwitchOff, "switch.off", eventHandler)
	subscribe(arrivalPresence, "presence.present", eventHandler)
	subscribe(departurePresence, "presence.not present", eventHandler)
	subscribe(smoke, "smoke.detected", eventHandler)
	subscribe(smoke, "smoke.tested", eventHandler)
	subscribe(smoke, "carbonMonoxide.detected", eventHandler)
	subscribe(water, "water.wet", eventHandler)
    subscribe(temperature, "temperature", eventHandler)
    subscribe(powerMeter, "power", eventHandler)
	subscribe(energyMeter, "energy", eventHandler)
    subscribe(signalStrength, "lqi", eventHandler)
    subscribe(signalStrength, "rssi", eventHandler)
	subscribe(button1, "button.pushed", eventHandler)
	if (triggerModes) {
		subscribe(location, modeChangeHandler)
	}
}

def getGXState(){
	childDevices*.refreshParty(4)
}
