DevMentor.Serialization
=======================



```cs
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Runtime.Serialization.Json;
using System.Text;
using System.Web;
using System.Xml;
using System.Xml.Serialization;

namespace DevMentor.Serialization
{
    public static class ObjectExtensions
    {

        public static void ToXmlFile(this object obj, string filePath)
        {
            try
            {
                using (Stream stream = File.Open(filePath, FileMode.Create, FileAccess.ReadWrite))
                {
                    XmlSerializer serializer = new XmlSerializer(obj.GetType());
                    XmlTextWriter writer = new XmlTextWriter(stream, Encoding.Default);
                    writer.Formatting = Formatting.Indented;
                    serializer.Serialize(writer, obj);
                    writer.Close();
                }
            }
            catch
            {
                throw;
            }
        }

        public static T FromXmlFile<T>(this string filePath)
        {
            try
            {
                XmlSerializer serializer = new XmlSerializer(typeof(T));
                T serializedData;
                using (Stream stream = File.Open(filePath, FileMode.Open, FileAccess.Read))
                {
                    serializedData = (T)serializer.Deserialize(stream);
                }
                return serializedData;
            }
            catch
            {
                throw;
            }
        }


        public static string ToJson(this object obj)
        {
            var jsonSerializer = new DataContractJsonSerializer(obj.GetType());
            string returnValue = "";
            using (var memoryStream = new MemoryStream())
            {
                using (var xmlWriter = JsonReaderWriterFactory.CreateJsonWriter(memoryStream))
                {
                    jsonSerializer.WriteObject(xmlWriter, obj);
                    xmlWriter.Flush();
                    returnValue = Encoding.UTF8.GetString(memoryStream.GetBuffer(), 0, (int)memoryStream.Length);
                }
            }
            return returnValue;
        }

        public static string ToXml(this object obj)
        {
            string result = string.Empty;
            try
            {
                using (var memoryStream = new MemoryStream())
                {
                    XmlSerializer serializer = new XmlSerializer(obj.GetType());
                    XmlTextWriter writer = new XmlTextWriter(memoryStream, Encoding.UTF8);
                    writer.Formatting = Formatting.Indented;
                    serializer.Serialize(writer, obj);
                    writer.Close();
                    result = UTF8ByteArrayToString(memoryStream.ToArray());
                }
            }
            catch
            {
                throw;
            }
            return result;
        }

        public static T FromXmlToObject<T>(this string xml)
        {
            T result = default(T);
            try
            {
                XmlSerializer xs = new XmlSerializer(typeof(T));
                MemoryStream memoryStream = new MemoryStream(StringToUTF8ByteArray(xml));
                XmlTextWriter xmlTextWriter = new XmlTextWriter(memoryStream, Encoding.UTF8);
                result = (T)xs.Deserialize(memoryStream);
            }
            catch
            {
                throw;
            }
            return result;
        }
        public static T FromJsonToObject<T>(this string json)
        {
            T returnValue;
            using (var memoryStream = new MemoryStream())
            {
                byte[] jsonBytes = Encoding.UTF8.GetBytes(json);
                memoryStream.Write(jsonBytes, 0, jsonBytes.Length);
                memoryStream.Seek(0, SeekOrigin.Begin);
                using (var jsonReader = JsonReaderWriterFactory.CreateJsonReader(memoryStream, 
                                          Encoding.UTF8, 
                                          XmlDictionaryReaderQuotas.Max, null))
                {
                    var serializer = new DataContractJsonSerializer(typeof(T));
                    returnValue = (T)serializer.ReadObject(jsonReader);

                }
            }
            return returnValue;
        }

        private static String UTF8ByteArrayToString(Byte[] characters)
        {
            UTF8Encoding encoding = new UTF8Encoding();
            String constructedString = encoding.GetString(characters);
            return (constructedString);
        }

        private static Byte[] StringToUTF8ByteArray(String pXmlString)
        {
            UTF8Encoding encoding = new UTF8Encoding();
            Byte[] byteArray = encoding.GetBytes(pXmlString);
            return byteArray;
        }
    }
}
```
