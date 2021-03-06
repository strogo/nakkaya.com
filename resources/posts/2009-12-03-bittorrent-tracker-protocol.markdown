---
title: BitTorrent Tracker Protocol
tags: bittorrent bencode clojure
---

Yesterday I got some free time and I thought I would complete another
piece from BitTorrent protocol. In order to download a torrent, you need
to communicate with the tracker and get a list of computers that are also
downloading the torrent you are interested in.

Tracker is an HTTP service that responds to GET requests. Our request
includes statistics about us that helps the tracker keep metrics about
the torrent. The response from the tracker includes a peer list and some
basic statistics about the
torrent. [Specs](http://wiki.theory.org/index.php/BitTorrentSpecification#Tracker_HTTP.2FHTTPS_Protocol)
are at theory.org and I used torrents from
[ubuntu](http://torrent.ubuntu.com:6969/) distribution for the tests.


If you want to jump to the code tracker.clj is
[here](/code/clojure/tracker.clj) bencode.clj is
[here](/code/clojure/bencode.clj)  in bencode.clj, two functions needs to
be replaced, to get correct SHA1 hashes. Replace the function below with
the ones in bencode.clj.

    (defn- decode-map [stream] 
      (let [list (decode-list stream)] 
        (with-meta 
          (apply hash-map list) 
          {:order (map first (partition 2 list))})))

    (defn- encode-dictionary [dictionary stream]
      (.write stream (int \d))
      (doseq [item (if (nil? (meta dictionary)) 
                     (keys dictionary)(:order (meta dictionary)))]
        (encode-object item stream)
        (encode-object (dictionary item) stream))
      (.write stream (int \e)))

The only difference with this one and the original is we keep the order
keys are read as meta data. The during encoding we encode according to
the order we read them, that way we get the original torrent file back
which will have a correct SHA1 hash. If you don't do this, map iteration
order will be different than the original and different hash will be
calculated.


Most important part of the request is the info hash part which
identifies which torrent we are interested in. It is the SHA1 hash of
the info dictionary not the whole file, in its bencoded form. That is
the reason we need to keep the order, same as the torrent file, if you
mess up the order hash will change.

    (defn calc-info-hash [torrent]
      (let [info (encode (torrent "info"))
            sha1 (MessageDigest/getInstance "SHA1")
            hash (.toString (BigInteger. (.digest sha1 info)) 16)
            pad (- 40 (count hash))]
        (str (apply str (take pad (repeat "0"))) hash)))

We are encoding the info dictionary using original order, then calculate the
SHA1 hash. SHA1 hash is a byte array to turn it in to hex, I used the
BigInteger class, there are lots of ways to do this I went with the
simplest. Pad the begging with 0s if it is less then 40 chars
long. Result is a hash such as the following,

    3e427a0a9d4826e76cd5363a004b2fa6baca1853

If you don't pay attention to the spec and send this directly to tracker
you will get an error this should be in [URL
Encoded](http://en.wikipedia.org/wiki/Percent-encoding) form. Padding
every two chars with % sign also doesn't work, been there done that
don't waste your time. Any hex in the hash that corresponds to a
unreserved character should be replaced, 

    a-z A-Z 0-9 -_.~

Partition the hex in to chunks of two and check if the hex corresponds
to any of these values, if they do replace them with the unreserved
char, 

    (defn url-encode [hash]
      (apply str
             (map (fn [[a b]] 
                    (let [byte (BigInteger. (str a b)  16) ]
                      (if (or (and (>= byte 65) (<= byte 90)) ; A-Z
                              (and (>= byte 97) (<= byte 122)) ; a-z
                              (and (>= byte 48) (<= byte 57)) ; 0-9
                              (= byte 45) (= byte 95) (= byte 46) (= byte 126))
                        (char byte) (str "%" a b)) )) (partition 2 hash))))

So that a hash such as,

    123456789abcdef123456789abcdef123456789a

becomes,

    %124Vx%9a%bc%de%f1%23Eg%89%ab%cd%ef%124Vx%9a

notice that hex 34 became 4 which is what it is in ASCII. You can test
the correctness of your hashes using the tracker url but don't request
from announce request from file,

    http://some.tracker.com/file?info_hash=hash

If you get a torrent back that means you have the correct hash.

Requests to announce requires all parameters to be set in order to be
valid, you can't build the request one by one, a working request should
include all of the following parameters.

    (defn build-request [torrent]
      (let [announce (torrent "announce")
            hash (calc-info-hash torrent)
            event "started"]
        (str announce "?" 
             "info_hash=" (url-encode hash) "&"
             "peer_id=" peer-id "&" "port=" port-in "&"
             "uploaded=0&downloaded=0&" "left=" ((torrent "info") "length") "&"
             "event=" event "&" "numwant=" peer-list-size "&compact=1" )))

Parameters are pretty much self explanatory, they are also explained in
the
[spec](http://wiki.theory.org/index.php/BitTorrentSpecification#Tracker_Request_Parameters). 

Tracker response is a bencoded dictionary containing statistics and
peers list. Peers value in the response map is a byte string multiple of
6, every 6 bytes  represent a peer. First 4 bytes contains the IP and last
two bytes contains the port they are listening on.

    (defn peers [peers]
      (reduce (fn[list peer]
                (conj list 
                      {:ip (apply str (interpose \. (map int (take 4 peer))))
                       :port (+ (* 256 (int (nth peer 5))) (int (nth peer 4)))}))
              [] (partition 6 peers)))

To parse this byte string we partition it in to 6 byte chunks, map first
4 bytes  to a integer to build the IP address, and use last two bytes to
build port number.

Now we have everything to extract some basic information from the
tracker,

    (defn get-torrent-stats [fname]
      (let [torrent (decode (FileInputStream. (File. fname)))
            request (request (build-request torrent))
            stats (decode (ByteArrayInputStream. (.getBytes request)))]
        {:complete (stats "complete") 
         :incomplete (stats "complete") 
         :peers (peers (stats "peers"))} ))

Putting it all together, and making the request should result it a map
containing the peers and statistics.

    tracker=> (get-torrent-stats "buntu.torrent")
    {:complete 0, :incomplete 0, 
     :peers [{:ip "84.29.195.139", :port 40898} 
             {:ip "195.153.89.87", :port 49768}]}
