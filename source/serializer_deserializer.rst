===========================
Serializers & DeSerializers
===========================

Serializers and DeSerializers are pluggable prestans constructs that assist the framework in packing or unpacking data. Serialzier or deserializer handle content of a particular mime type and are generally wrappers to vocabularies already available in Python (although it possible to write a custom serializer entirely in Python). There are two types of serializers and deserializers:

* ``Textual`` - serialization formats that are text based (e.g JSON, XML, CSV) and are the ones that applications common use to communicate
* ``Binary`` - serialization formats that are binary (e.g PDF) and are mostly used as an output format.

prestans application may speak as many vocabularies as they wish; vocabularies can also be local to handlers (as opposed to applicaiton wide). You must also define a default format.

Each request must send an ``Accept`` header for prestans to decide the response format. If the registered handler cannot respond in the requeted format prestans raises an ``UnsupportedVocabularyError`` exception inturn producing a ``501 Not Implemented`` response. All prestans APIs have a set of default formats all handlers accept, each end-point might accept additional formats.

If a request has send a body (e.g ``PUT``, ``POST``) you must send a ``Content-Type`` header to declare the format in use. If you do not send a ``Content-Type`` header prestans will attempt to use the default deserializer to deserialize the body. If the ``Content-Type`` is not supported by the API an ``UnsupportedContentTypeError``` exception is raised inturn producing a ``501 Not Implemented`` response.

