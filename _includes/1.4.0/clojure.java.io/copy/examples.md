### Example 0
[permalink](#example-0)

{% highlight clojure linenos %}
{% raw %}
(ns your-project
  (:require [clojure.java.io :as io]))

(defn copy-file [source-path dest-path]
  (io/copy (io/file source-path) (io/file dest-path)))

(copy-file "/home/username/squirrel.txt" "/home/username/burt-reynolds.txt"){% endraw %}
{% endhighlight %}


