# PlayOn
A script in Python 3 to play, and control through a web interface, local and remote contents to a DLNA/UPnP renderer

PlayOn is an application written in Python 3 designed to play media contents on a DLNA/UPnP renderer from a computer running under Windows, and to allow control through a web interface from any device connected to the network. The script does not need any other package, but to enable all the features, ffmpeg and youtube-dl must be installed.
The application has been tested on a Samsung UE40F8000 TV, but should work on any DLNA compliant renderer, except for some manufacturer/model dependent protocols, such as subtitle instructions.

The usable media sources are:
- local files (on the computer where the application is running)
- remote files from the local network or from internet
- content from a webpage as long as its address can be extracted by youtube-dl; they can be in a single stream or in separate video and audio streams (such as Youtube high resolution contents)

The application runs a server that can operate in two modes (except if 'typeserver' is set as 'n': the URI will directly be sent to the renderer):
- a random mode, where the content is loaded into a buffer to be distributed to the renderer, and time seeking is allowed during playing from the renderer or from the web interface (provided the source can be accessed in random mode)
- a sequential mode, where the content can be muxed/remuxed in live time with ffmpeg to be delivered, and no time seeking is possible, but a start position can be defined: HLS and video+audio contents can this way be sent to the renderer, which must be able to play either fMp4 or MpegTS streams

To install the application:
- of course, install Python 3
- copy PlayOn.py, ffmpeg.bat and youtube-dl.bat in the same folder
- install ffmpeg (https://ffmpeg.zeranoe.com/builds/) and youtube-dl (https://youtube-dl.org/)
- open ffmpeg.bat and put the location of ffmpeg executable in the first line (more customization can be made later on)
- open youtube-dl.bat and put the location of youtube-dl executable in the first line (more customization can be made later on)
- allow ffmpeg and python to communicate through the firewall (for more precise needs, see below)

To simply launch the application: PlayOn s
The play session can be controlled from any device on the same network by opening in a browser the address of the web interface.

To launch the application with more options (PlayOn -h to display the complete syntax of command line and abbreviated commands):
- to only diplay the available renderers: PlayOn.py display_renderers [-h] [--ip SERVER_IP_ADDRESS] [--port SERVER_TCP_PORT] [--verbosity VERBOSE]
- to open the web launch page to select the renderer, enter the content address and start playing: PlayOn.py start [-h] [--ip SERVER_IP_ADDRESS] [--port SERVER_TCP_PORT] [--typeserver TYPE_SERVER] [--buffersize BUFFER_SIZE] [--bufferahead BUFFER_AHEAD] [--muxcontainer MUX_CONTAINER] [--mediasrc MEDIA_ADDRESS] [--mediasubsrc MEDIA_SUBADDRESS] [--mediasublang MEDIA_SUBLANG] [--mediastartfrom MEDIA_START_FROM] [--verbosity VERBOSE]
- to directly start playing a content and open the web control page: PlayOn.py control [-h] [--ip SERVER_IP_ADDRESS] [--port SERVER_TCP_PORT] [--typeserver TYPE_SERVER] [--buffersize BUFFER_SIZE] [--bufferahead BUFFER_AHEAD] [--muxcontainer MUX_CONTAINER] [--uuid RENDERER_UUID] [--name RENDERER_NAME] [--mediasubsrc MEDIA_SUBADDRESS] [--mediasublang MEDIA_SUBLANG] [--mediastartfrom MEDIA_START_FROM] [--verbosity VERBOSE] MEDIA_ADDRESS  
where:  
  --ip SERVER_IP_ADDRESS, -i SERVER_IP_ADDRESS            IP address to be used for the web server [default: computer address on the network]  
  --port SERVER_TCP_PORT, -p SERVER_TCP_PORT              TCP port to be used for the web server [default: 8000]  
  --typeserver TYPE_SERVER, -t TYPE_SERVER                server type (a:auto, s:sequential, r:random, n:none) [default: a]  
  --buffersize BUFFER_SIZE, -b BUFFER_SIZE                buffer size in MB [default: 75]  
  --bufferahead BUFFER_AHEAD, -a BUFFER_AHEAD             load ahead buffer size in MB [default: 25]  
  --muxcontainer MUX_CONTAINER, -m MUX_CONTAINER          remux container type, preceded by ! for systematic remux [default: MP4]  
  --uuid RENDERER_UUID, -u RENDERER_UUID                  uuid of the renderer [default: first renderer found]  
  --name RENDERER_NAME, -n RENDERER_NAME                  renderer name [default: first renderer found]  
  --mediasrc MEDIA_ADDRESS, -c MEDIA_ADDRESS              optional content address [default: none]  
  MEDIA_ADDRESS                                           required content address  
  --mediasubsrc MEDIA_SUBADDRESS, -s MEDIA_SUBADDRESS     subtitle content address [default: none]  
  --mediasublang MEDIA_SUBLANG, -l MEDIA_SUBLANG          subtitle prefered language, . for no selection [default: fr,fre,fra]  
  --mediastartfrom MEDIA_START_FROM, -f MEDIA_START_FROM  time position to strat from in format H:MM:SS [default: beginning]  
  --verbosity VERBOSE, -v VERBOSE                         verbosity level from 0 to 2 [default: 2]  
  
Exemples (let's suppose the IP address of the computer is 192.168.1.10):
- PlayOn s -p 9000 -t s -m mpegts: will start PlayOn in "start" mode, with the server in "sequential" mode, the mux optional in mpegts, and the web interface can be reached on http://192.168.1.10:9000
- PlayOn c -t r https://www.youtube.com/watch?v=XXXXXXX: will play the YT video in "random" mode (and thus in the best resolution available as single stream)
- PlayOn c -m mp4 https://www.youtube.com/watch?v=XXXXXXX: will play the YT video in the best resolution available, and probably in sequential mode, as there will be a video and an audio stream to mux
- PlayOn c C:\video.mkv: will play the local file in "random" mode, with its subtitles if the file contains some, or an external subtitle file with the same name is in the same folder
- PlayOn c -t s C:\video.mkv -s C:\video.mkv: will play the local file in "sequential" mode, with its subtitles if the file contains some
- PlayOn c https://www.youtube.com/watch?v=XXXXXXX -s https://www.youtube.com/watch?v=XXXXXXX -l en: will play the YT video with its subtitles in English

Tips to make the application easier to use:
- in %appdata%\Microsoft\Windows\SendTo, create a shortcut with 'C:\Windows\py.exe "C:\...\PlayOn.py" c -n "[TV]Samsung LED40"' where obviously '...' must be replaced with the path to the script, and '[TV]Samsung LED40' by the name of the renderer
- in Firefox or Chrome, install the add-on called 'Open With' and create a shortcut with py '"C:\...\PlayOn.py" c -n "[TV]Samsung LED40" "%s "'
- in Firefor or Chrome, install a QR code generation add-on, such as 'QR Code (Generator and Reader)', and once the control page opened on the computer, generate and flash the QR code with a smartphone to be able to control the session from your sofa

The application will altern between the "start" and the "control" page when a session begins or ends.
In the "start" page, the subtitile address can be entered as a second line in the field.
In the "control" page, the seekbar will be displayed if time seek is possible (only in random server mode provided that the source of the content supports it).
From the command line window, it is possible to cycle between the muxcontainers and the types of server by pressing the key indicated on the screen ('m' to swith between 'fmp4' and 'mpegts', '!' to switch between facultative and required remuxing, 't' to switch between automatic, sequential and random modes for the server); to exit the application, press 's'.

Some more infos:
- if the server is in sequential mode muxcontainer can be "fmp4" or "mpegts"; depending on the source, one or the other format will give better results, in terms, in particular, of synchronization (mpegts tends to be more accurate for subtitles as it is set in ffmpeg.bat)
- if the server is in sequential mode, if remux is optional (no "!"), it will be used only if a start position is set or if the source is made of two streams or if it is in HLS (m3u8)
- the buffersize less the buffersizeahead should exceed the buffer of the renderer for a smoother experience when the server is in random mode (to be able to move backwards a few seconds without having to reload the content)
- if the server mode is on 'auto', 'random' will be choosed for local contents except if remux is required ('!'), and for network contents, except, in addition of this case, if the server does not support partial requests or also, for video sites, if the content is available in better resolution in video and audio separate streams
- if the server mode is on 'sequential' and the content is available in a higher resolution in two streams, this choice will be made only if remux is required ("!")
- in 'random' mode, for local content, the application will look for an external subtitle file with the same name that the file (and the appropriated extension)
- in 'sequential' mode, to play a content containing subtitles, pass the address both as main and subtitle content, or type it twice in the "start" page URL field
- in the firewall, ffmpeg needs to be set with outgoing TCP connections allowed, and python with outgoing TCP and UDP connections, incoming TCP connections from local network on local ports in the range of SERVER_TCP_PORT (as in command line) and SERVER_TCP_PORT+9, incoming UDP connections from local network on local port 1900, all allowed
- more customization can be done in ffmpeg.bat and youtube-dl.bat, possibly on a per renderer basis as the renderer name is available through "!mediabuilder_profile!": for example, in the provided youtube-dl.bat file, there is a generic configuration, but also specific configurations for the TV and the phone, to select the right streams according the the capacities of the renderer (the syntax to be used is available on youtube-dl page), and in the provided ffmpeg.bat file, there is only a generic configuration, that could be customized by renderer if needed or to add codec conversion in addition to remux (but is must be done in real time)
- the other scripts, DLNAControler and MediaServer, can be used to either explore the features of the renderers, or directly run the content server 

What remains to be done:
- improve the web interface appearance
- translate in other languages than French
- add wider compatibility with renderers of other model and manufacturers (requires packet sniffing as documentation usually is scarce)

Enjoy !