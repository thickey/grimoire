{% highlight clojure linenos %}
(deftype AsmReflector [class-resolver]
  Reflector
  (do-reflect [_ typeref]
    (with-open [is (resolve-class class-resolver typeref)]
      (let [class-symbol (typesym typeref)
            r (ClassReader. is)
            result (atom {:bases #{} :flags #{} :members #{}})]
        (.accept
         r
         (reify
          ClassVisitor
          (visit [_ version access name signature superName interfaces]
                 (let [flags (parse-flags access :class)
                       ;; ignore java.lang.Object on interfaces to match reflection
                       superName (if (and (flags :interface)
                                          (= superName "java/lang/Object"))
                                   nil
                                   superName)
                       bases (->> (cons superName interfaces)
                                  (remove nil?)
                                  (map internal-name->class-symbol)
                                  (map symbol)
                                  (set)
                                  (not-empty))]
                   (swap! result merge {:bases bases 
                                        :flags flags})))
          (visitSource [_ name debug])
          (visitInnerClass [_ name outerName innerName access])
          (visitField [_ access name desc signature value]
                      (swap! result update-in [:members] (fnil conj #{})
                             (Field. (symbol name)
                                     (field-descriptor->class-symbol desc)
                                     class-symbol
                                     (parse-flags access :field)))
                      nil)
          (visitMethod [_ access name desc signature exceptions]
                       (when-not (= name "<clinit>")
                         (let [constructor? (= name "<init>")]
                           (swap! result update-in [:members] (fnil conj #{})
                                  (let [{:keys [parameter-types return-type]} (parse-method-descriptor desc)
                                        flags (parse-flags access :method)]
                                    (if constructor?
                                      (Constructor. class-symbol
                                                    class-symbol
                                                    parameter-types
                                                    (vec (map internal-name->class-symbol exceptions))
                                                    flags)
                                      (Method. (symbol name)
                                               return-type
                                               class-symbol
                                               parameter-types
                                               (vec (map internal-name->class-symbol exceptions))
                                               flags))))))
                       nil)
          (visitEnd [_])
          ) 0)
        @result))))
{% endhighlight %}
