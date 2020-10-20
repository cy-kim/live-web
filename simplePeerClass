class SimplePeerWrapper {
    constructor(initiator, socket_id, socket, stream) {

        this.simplepeer = new SimplePeer({
            initiator: initiator,
            trickle: false
        });


        // Their socket id, our unique id for them
        this.socket_id = socket_id;

        // Socket.io Socket
        this.socket = socket;

        // Our video stream - need getters and setters for this --local stream
        this.stream = stream;

        // Initialize mediaStream to null
        this.peerStream = null;

        this.volume = 0;
        this.mappedVolume = 0;

        this.newVolume =null;

        // simplepeer generates signals which need to be sent across socket
        this.simplepeer.on('signal', data => {
            this.socket.emit('signal', this.socket_id, this.socket.id, data);
        });

        // When we have a connection, send our stream
        this.simplepeer.on('connect', () => {
            console.log('CONNECT')
            //console.log(this.simplepeer);

            // Let's give them our stream
            this.simplepeer.addStream(stream);

            console.log("Send our stream");
        });

        // Stream coming in to us
        this.simplepeer.on('stream', stream => {
            //console.log('Incoming Stream');
            console.log("Incoming stream" , stream);
            console.log(stream.getAudioTracks());

            this.peerStream = stream

            audioContext = new(window.AudioContext || window.webkitAudioContext)();

            playStream (this.peerStream, this.volume);

            let newVolume = document.createElement("p");
            newVolume.id = this.socket_id;
            newVolume.zIndex = 1000;
            document.body.appendChild(newVolume);

            mediaStreamSource = audioContext.createMediaStreamSource(this.peerStream);
            meter = this.createAudioMeter(audioContext);
            mediaStreamSource.connect(meter);



        });

    }

    //https://codepen.io/huooo/pen/xJNPOL
    createAudioMeter(audioContext, clipLevel, averaging, clipLag){
        const processor = audioContext.createScriptProcessor(512);
        processor.onaudioprocess = this.volumeAudioProcess;
        processor.clipping = false;
        processor.lastClip = 0;
        processor.volume = 0;
        processor.clipLevel = clipLevel || 0.98;
        processor.averaging = averaging || 0.95;
        processor.clipLag = clipLag || 750;

        processor.connect(audioContext.destination)

        processor.checkClipping = function(){
            if(!this.clipping){
                return false;
            }
            if((this.lastClip + this.clipLag)< window.performance.now()){
                this.clipping = false;
            }
            return this.clipping
        }

        processor.shutdown = function(){
            this.disconnect()
            this.onaudioprocess = null;
        } 

        return processor
    }

    volumeAudioProcess(event) {
        const buf = event.inputBuffer.getChannelData(0)
        const bufLength = buf.length
        let sum = 0
        let x

        // Do a root-mean-square on the samples: sum up the squares...
        for (var i = 0; i < bufLength; i++) {
        x = buf[i]
        if (Math.abs(x) >= this.clipLevel) {
            this.clipping = true
            this.lastClip = window.performance.now()
        }
        sum += x * x
        }

        // ... then take the square root of the sum.
        const rms = Math.sqrt(sum / bufLength)

        // Now smooth this out with the averaging factor applied
        // to the previous sample - take the max here because we
        // want "fast attack, slow release."
        this.volume = Math.max(rms, this.volume * this.averaging)
        this.mappedVolume = Math.floor(mapRange(this.volume, 0, 1, 0, 50));


        //document.getElementById(this.socket_id).innerHTML = this.mappedVolume;
        //console.log(this.socket_id);


    }

    inputsignal(sig) {
         this.simplepeer.signal(sig);
     }

}
