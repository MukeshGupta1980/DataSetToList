# DataSetToList
Convert Data Set to list in few lines of code

// See https://aka.ms/new-console-template for more information

using System.Data;
using System.Reflection;
using System.Reflection.PortableExecutable;

// Define Column Name in Data Table
string[] headers = ["T1_Name","T2_Name","T3_Name","T4_Name", "T5_Name"];
DataTable dt = new DataTable();
// Adding Column in Table
foreach (string strHeader in headers)
{
    dt.Columns.Add(strHeader);
}
// Ading Data in Table
string[] data = ["Data1", "Data2","TESTT"];
foreach (string strHeader in data)
{
    dt.Rows.Add(strHeader, strHeader, 123,DateTime.Now);
}

MyExtract myClass = new MyExtract();
List<MyClass> myClass2 = new List<MyClass>();
myClass2 = myClass.DataTableToList<MyClass>(dt);
string temp= Newtonsoft.Json.JsonConvert.SerializeObject(myClass2);
Console.ReadLine();


namespace RandD
{
    
    public class MyCustomAttribute : Attribute
    {
        public string ColumnName { get; set; }
    }
    public class MyExtract
    {
        public List<T> DataTableToList<T>(DataTable table) where T : new()
        {
            List<T> list = new List<T>();

            var typeProperties = typeof(T).GetProperties(BindingFlags.Public | BindingFlags.Instance).Select(propertyInfo => new
            {
                PropertyInfo = propertyInfo,
                Type = Nullable.GetUnderlyingType(propertyInfo.PropertyType) ?? propertyInfo.PropertyType
            }).ToList();

            foreach (var row in table.Rows.Cast<DataRow>())
            {
                T obj = new T();
                PropertyInfo[] properties = typeof(T).GetProperties();
                foreach (PropertyInfo typeProperty in properties)
                {
                    MyCustomAttribute[] keys = (MyCustomAttribute[])typeProperty.GetCustomAttributes(typeof(MyCustomAttribute), true);

                    if (keys.Length > 0)
                    {
                        MyCustomAttribute headerinfo = keys[0];
                        object value = row[headerinfo.ColumnName];
                        if (value != null)
                        {
                            object safeValue = Convert.ChangeType(value, typeProperty.PropertyType);
                            typeProperty.SetValue(obj, safeValue, null);
                        }



                    }
                }
                list.Add(obj);
            }
            return list;
        }
    }
        public class MyClass
    {
        // Column Name Exist in Data Table Column Name   
         [MyCustomAttribute(ColumnName= "T1_Name")]
        public string T1 { get; set; }

        [MyCustomAttribute(ColumnName = "T2_Name")]
        public string T2 { get; set; }

        [MyCustomAttribute(ColumnName = "T3_Name")]
        public int T3 { get; set; }

        [MyCustomAttribute(ColumnName = "T4_Name")]
        public string T4 { get; set; }

        public string T5 { get; set; }

    }

}
