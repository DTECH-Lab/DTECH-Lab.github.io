<head>
<script>
window.AudioContext = window.AudioContext || window.webkitAudioContext;
var context = new AudioContext();

function audioBufferToText(audioBuffer, normalize, resample, sampleRate)
{
	var buffer = audioBuffer.getChannelData(0);
		
	if(audioBuffer.numberOfChannels > 1)
	{
		for(var c = 1; c < audioBuffer.numberOfChannels; c++)
		{
			var cb = audioBuffer.getChannelData(c);
			for(var i = 0; i < length; i++)
				buffer[i] += cb[i];
		}
	}
	
	if(resample)
	{
		var scale = audioBuffer.sampleRate / sampleRate;
		var length = Math.floor((buffer.length - 1) / scale); 
		var b = new Float32Array(length);
		for(var i = 0; i < length; i++)
		{
			p = Math.floor(i * scale);
			b[i] = buffer[p];
		}
		buffer = b;
	}
	else
		sampleRate = audioBuffer.sampleRate;
				
	var text = "const unsigned int sampleRate = " + sampleRate + ";\r\n";
	text += "const unsigned int sampleCount = " + buffer.length + ";\r\n";
	text += "const signed char samples[] = {"

	var max = 0;
	if(normalize)
		for(var i = 0; i < buffer.length; i++)
			max = Math.max(Math.abs(buffer[i]), max);
	if(max == 0) max = 1;
	for(var i = 0; i < buffer.length; i++)
	{
		if((i & 15) == 0) text += "\r\n";
		text += Math.round(buffer[i] / max * 127) + ", ";
	}	
	text += "};\r\n";
	return text;
}


function enrickSound(buffer)
{
	//if you are looking for this function.. sorry that was a gag
}

function convert(event)
{
	var audioBuffer = null;
	var reader = new FileReader();
	var file = event.target.files[0];
	reader.onload = function(){
	    context.decodeAudioData(reader.result, function(buffer) {
			var link = document.createElement("a");
			link.download = file.name.split('.', 1)[0] + ".h";
			link.href = URL.createObjectURL(new Blob(
			[audioBufferToText(buffer, document.getElementById("normalize").checked, document.getElementById("resample").checked,                                                   document.getElementById("samplerate").value)], {type: "text/plain"}));
			document.body.appendChild(document.createElement("br"));
			document.body.appendChild(link);
			link.innerHTML = link.download;
			link.click();
			//document.body.removeChild(link);
			event.target.value = "";
		}, function(){
			//error
		});
	}
	reader.readAsArrayBuffer(file);
}
</script>
</head>
<body style="font-family: arial">
<h1> audio to header converter</h1>
Open an audio file supported by the browser to convert it to a c++ header file (8Bit signed).<br><br>
<input type="file" onchange="convert(event)"><br><br>
Export: <input id="normalize" type="checkbox" checked>normalize <input id="resample" type="checkbox">resample <input id="samplerate" value="11025"><br>

	
	
	
