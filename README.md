# xsxml

The embedded xml SAX parser, extract from rapidxml DOM parser: https://github.com/dwd/rapidxml  
The non-recursive revision is extract from pugixml DOM parser: https://github.com/zeux/pugixml

## SAX2 handler
```cpp
///////////////////////////////////////////////////////////////////////////
// Implement XML SAX2 handler with SAX3 handler
class xmlSAX2Handler
{
    xsxml::xml_sax3_parse_cb sax3_handler_;
public:
    xmlSAX2Handler() { 
        sax3_handler_.xml_start_element_cb = [=](char* name, size_t size) {
            elementName.first = name;
            elementName.second = size;
        };
        sax3_handler_.xml_attr_cb = [=](const char* name, size_t,
            const char* value, size_t) {
                elementAttrs.push_back(name);
                elementAttrs.push_back(value);
        };
        sax3_handler_.xml_end_element_cb = [=](const char* name, size_t len) {
            auto chTemp = elementName.first[elementName.second];
            elementName.first[elementName.second] = '\0';

            if (!elementAttrs.empty()) {
                elementAttrs.push_back(nullptr);
                xmlSAX2StartElement(elementName.first, elementName.second, &elementAttrs[0], elementAttrs.size() - 1);
                elementAttrs.clear();
            }
            else {
                const char* attr = nullptr;
                const char** attrs = &attr;
                xmlSAX2StartElement(elementName.first, elementName.second, attrs, 0);
            }

            elementName.first[elementName.second] = chTemp;

            xmlSAX2EndElement(name, len);
        };
        sax3_handler_.xml_text_cb = [=](const char* s, size_t len) {
            xmlSAX2Text(s, len);
        };
        elementAttrs.reserve(64); 
    }

    /**
    * @remark: The parameter 'name' without null terminator charactor
    */
    virtual void xmlSAX2StartElement(const char* name, size_t, const char** atts, size_t) = 0;

    /**
    * @remark: The parameter 'name' has null terminator charactor
    */
    virtual void xmlSAX2EndElement(const char* name, size_t) = 0;
    /**
    * @remark: The parameter 's' has null terminator charactor
    */
    virtual void xmlSAX2Text(const char* s, size_t) = 0;

private:
    xsxml::string_view elementName;
    std::vector<const char*> elementAttrs;
};
```
