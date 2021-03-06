# mongoDbDemo
1.install below dll by nuget .
MongoDB.Bson
MongoDB.Driver
MongoDB.Driver.Core

2.add below demo code.
using MongoDB.Bson;
using MongoDB.Driver;
using System;
using System.Collections.Generic;

namespace MongoDBTest_CoreConsole
{
    class Program
    {
        static void Main(string[] args)
        {
            var client = new MongoClient("mongodb://localhost:27017");
            var database = client.GetDatabase("mongoDbTest");
            var collection = database.GetCollection<BsonDocument>("student");

            var collection9 = database.GetCollection<Userinfo>("userinfos");
            var collection1 = database.GetCollection<BsonDocument>("userinfos");

            ////待添加的document
            //var doc9 = new BsonDocument{
            //            { "_id",7 },
            //            { "name", "吴九" },
            //            { "age", 29 },
            //            { "ename", new BsonDocument
            //                {
            //                    { "firstname", "jiu" },
            //                    { "lastname", "wu" }
            //                }
            //            }
            //        };
            //collection1.InsertOne(doc9);



            //Fileter用于过滤，如查询name = 吴九的第一条记录
            var filter = Builders<BsonDocument>.Filter;
            //Find(filter)进行查询
            var doc = collection.Find(filter.Eq("name", "MongoDB")).FirstOrDefault();
            if(doc==null)
            {
                var document = new BsonDocument
                                        {
                                            { "name", "MongoDB" },
                                            { "type", "Database" },
                                            { "count", 1 },
                                            { "info", new BsonDocument
                                                {
                                                    { "x", 203 },
                                                    { "y", 102 }
                                                }}
                                        };
                collection.InsertOne(document);

                 //待添加的document
                 var doc1 = new BsonDocument{
                        { "_id",7 },
                        { "name", "吴九" },
                        { "age", 29 },
                        { "ename", new BsonDocument
                            {
                                { "firstname", "jiu" },
                                { "lastname", "wu" }
                            }
                        }
                    };
                collection.InsertOne(doc1);

                //待添加的document
                var doc2 = new BsonDocument{
                        { "_id",7 },
                        { "name", "吴九" },
                        { "age", 29 },
                        { "ename", new BsonDocument
                            {
                                { "firstname", "jiu" },
                                { "lastname", "wu" }
                            }
                        }
                    };
                collection.InsertOne(doc2);

                doc = collection.Find(filter.Eq("name", "MongoDB")).FirstOrDefault();
            }

            Console.WriteLine(doc.ToString());
            var filter1 = Builders<BsonDocument>.Filter;
            var docs = collection.Find(filter1.Eq("name", "MongoDB") & filter.Eq("count", 1)).ToList();
            docs.ForEach(d => Console.WriteLine(d));

            var filter2 = Builders<BsonDocument>.Filter;
            var docs2 = collection.Find(filter2.Lt("age", 25) | filter2.Gt("age", 28)).ToList();
            docs2.ForEach(d => Console.WriteLine(d));

            //查询存在address字段的记录
            var filter3 = Builders<BsonDocument>.Filter;
            var docs3 = collection.Find(filter3.Exists("address")).ToList();
            docs3.ForEach(d => Console.WriteLine(d));

            //查询age<26的记录，按年龄倒序排列
            var filter5 = Builders<BsonDocument>.Filter;
            var sort5 = Builders<BsonDocument>.Sort;
            var docs5 = collection.Find(filter5.Lt("age", 26))//过滤
                                 .Sort(sort5.Descending("age")).ToList();//按age倒序
            docs5.ForEach(d => Console.WriteLine(d));


            //查询age<26的记录 包含name age 排除 _id
            var project6 = Builders<BsonDocument>.Projection;
            var filter6 = Builders<BsonDocument>.Filter;
            var docs6 = collection.Find(filter6.Lt("age", 26))//过滤
                                 .Project(project6.Include("name")//包含name
                                                 .Include("age")//包含age
                                                 .Exclude("_id")//不包含_id
                                       ).ToList();
            docs6.ForEach(d => Console.WriteLine(d));
            
            var filter7 = Builders<BsonDocument>.Filter;
            var update7 = Builders<BsonDocument>.Update;
            var project7 = Builders<BsonDocument>.Projection;
            //将所有年龄小于25的记录标记为young（如果没有mark字段会自动添加）
            UpdateResult reulst = collection.UpdateMany(filter.Lt("age", 25), update7.Set("mark", "young"));
            if (reulst.IsModifiedCountAvailable)
            {
                Console.WriteLine($"符合条件的有{reulst.MatchedCount}条记录");
                Console.WriteLine($"一共修改了{reulst.ModifiedCount}条记录");
                //查询修改后的记录
                var docs7 = collection.Find(filter7.Empty)
                                .Project(project7.Include("age").Include("name").Include("mark"))
                                .ToList();

                docs7.ForEach(d => Console.WriteLine(d));
            }


            //删除名字为张三的记录
            //var filter8 = Builders<BsonDocument>.Filter;
            //var project8 = Builders<BsonDocument>.Projection;
            //collection.DeleteOne(filter.Eq("name", "张三"));
            //var docs8 = collection.Find(filter8.Empty)
            //                .Project(project8.Include("age").Include("name").Include("mark"))
            //                .ToList();
            //docs8.ForEach(d => Console.WriteLine(d));


            ///类型映射
            var filter9 = Builders<Userinfo>.Filter;
            var sort9 = Builders<Userinfo>.Sort;
            List<Userinfo> userinfos = collection9.Find(filter9.Gt("age", 25))   //查询年龄小于25岁的记录
                                                 .Sort(sort9.Descending("age"))  //按年龄进行倒序
                                         .ToList();
            //遍历结果
            userinfos.ForEach(u => Console.WriteLine($"姓名：{u.name},年龄：{u.age},英文名：{u.ename.firstname} {u.ename.lastname}"));



            Console.WriteLine("Hello World!");
            Console.ReadKey();
        }
    }

    /// <summary>
    /// 用户类
    /// </summary>
    public class Userinfo
    {
        public int _id { get; set; }//id
        public string name { get; set; }//姓名
        public int age { get; set; }//年龄
        public int level { get; set; }//等级
        public Ename ename { get; set; }//英文名
        public string[] roles { get; set; }//角色
        public string address { get; set; }//地址
    }

    /// <summary>
    /// 英文名
    /// </summary>
    public class Ename
    {
        public string firstname { get; set; }
        public string lastname { get; set; }
    }
}
