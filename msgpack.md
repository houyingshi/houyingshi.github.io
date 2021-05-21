MsgPack是一个简单高效的序列化框架。

通过以下代码可以了解到

1.通过数组索引代替键值对，有效的减小数据大小；

2.类中字段按照自母排序序列化；

3.父类和子类的所有字段按照字母排序进行序列化；

4.字段按照字母排序，会导致后续类扩展的兼容性问题；
using MsgPack;
using MsgPack.Serialization;
using System;
using System.Collections.Generic;
using System.IO;

namespace ConsoleApp1
{
    public class Person
    {
        public string Name { get; set; }
        public int Age { get; set; }

        public Person Father { get; set; }

        public List<Person> Teachers { get; set; }

        public IDictionary<string, Person> Courses { get; set; }
    }

    public class Worker:Person
    {
        public double Salary { get; set; }
    }

    public class Program
    {
        public static void Main(string[] args)
        {
            var person = new Person()
            {
                Name = "mango",
                Age = 10,
                Father = new Person()
                {
                    Name = "father"
                },
                Courses = new Dictionary<string, Person>() {
                    { "math", new Person() { Name= "math teacher" } }
                },
                Teachers = new List<Person>() {
                    new Person() { Name= "english teacher" }
                }
            };

            var worker = new Worker()
            {
                Name = "worker",
                Age = 30,
                Father = new Person()
                {
                    Name = "father"
                },
                Courses = new Dictionary<string, Person>() {
                    { "math", new Person() { Name= "math teacher" } }
                },
                Teachers = new List<Person>() {
                    new Person() { Name= "english teacher" }
                },
                Salary = 10000
            };

            var data = SerializePerson(person);
            ShowPersonPackStructure(data);
            ShowWorkerPackStructure(worker);
            UsePackerPack();

            
            Console.Read();
        }

        static byte[] SerializePerson(Person person)
        {
            var serializer = MessagePackSerializer.Get<Person>();
            var data = Pack<Person>(serializer, person);
            NewtonSerialize(person);
            return data;
        }

        static void ShowPersonPackStructure(byte[] data)
        {
            ExploreMsgPackStructure(data, typeof(Person).Name);
        }

        static void ShowWorkerPackStructure(Worker worker)
        {
            var serializer = MessagePackSerializer.Get<Worker>();
            var data = Pack(serializer, worker);
            ExploreMsgPackStructure(data, worker.GetType().Name);
        }

        static void UsePackerPack()
        {
            var stringData = Pack();
            ExploreMsgPackStructure(stringData, " Packer");
        }

        static void NewtonSerialize(Person person)
        {
            var jsonObj = Newtonsoft.Json.JsonConvert.SerializeObject(person);
            var jsonData = System.Text.Encoding.Default.GetBytes(jsonObj);
            Console.WriteLine("json serialize "+ person .GetType().Name+ " size is " + jsonData.Length);
        }

       static byte[] Pack<T>(MessagePackSerializer<T> serializer,  object obj)
        {
            using (var stream = new MemoryStream())
            {
                serializer.Pack(stream, obj);
                using (FileStream fs = new FileStream(obj.GetType().Name + ".txt", FileMode.OpenOrCreate))
                {
                    var data = stream.ToArray();
                    Console.WriteLine("msgpack packed "+ obj.GetType().Name + " size is " + data.Length);
                    fs.Write(data, 0, data.Length);
                    return data;
                }
            }
        }

        static byte[] Pack()
        {
            using (var stream = new MemoryStream())
            {
                var packer = Packer.Create(stream);
                packer = packer.PackString("mango");
                packer = packer.PackString("is");             
                packer = packer.Pack(10);
                packer = packer.PackString("years");
                packer = packer.PackCollection<string>(new List<string>() { "this is a list"});
                var data = stream.ToArray();
                return data;
            }
        }

        static void ExploreMsgPackStructure(byte[] packData, string objectName)
        {
            Console.WriteLine("msgpack  "+ objectName+" structure is :");
            int position = 0;
            while (position < packData.Length)
            {
                var obj = Unpacking.UnpackObject(packData, position);
                Console.WriteLine(obj.Value);
                position += obj.ReadCount;
            }
        }
    }


}
