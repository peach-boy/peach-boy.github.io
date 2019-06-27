---
title: java8之lamda表达式
date: 2019-06-18 16:47:39
categories: java基础
tags:
---

## lamda表达式 ##
java8中lamda表达式的引入，标志着java开始拥抱主流函数式编程语言，其实lamda表达式并不是什么新鲜事物，在JavaScript中早已存在。作为经典语言引入lamda表达式以及java8中的其他新特性，如流式处理，会一定程度上让代码更简洁，当然还是无法和JavaScript这类语言相比，但这也正是各自所处的角色的不同决定的。
## lamda表达式的格式 ##
1. 参数类型声明：可以不需要声明参数类型，编译器会识别参数值。
2. 参数圆括号（可选）：在单个参数时可以不使用括号，多个参数时必须使用。
3. 大括号和return关键字（可选）：如果只有一个表达式，则可以省略大括号和return关键字，编译器会自动的返回值；相对的，在使用大括号的情况下，则必须指明返回值。

## lamda表达式例子 ##
以Collections.sort为例

    /**
     * @author xiantao.wu
     * @create 2018/8/1514:09
     **/
    public class TestLamda {
        public static void main(String[] args) {
            //原生方法
            List<Student> students1=getStudentList();
            Collections.sort(students1, new Comparator<Student>() {
                @Override
                public int compare(Student o1, Student o2) {
                    return o1.getAge().compareTo(o2.getAge());
                }
            });
            students1.forEach(in-> System.out.print(in.getName()+in.getAge()+"|"));
            System.out.println("原生方法");
    
    
            //第一种情况：传入静态方法
            List<Student> students2=getStudentList();
            Collections.sort(students2, (s1, s2) -> Student.sortByAgeStatic(s1, s2));
            students2.forEach(in-> System.out.print(in.getName()+in.getAge()+"|"));
            System.out.println("第一种情况：传入静态方法");
    
    
            //第二种情况：传入实例方法
            List<Student> students3=getStudentList();
            Collections.sort(students3, (s1, s2) -> new Student().sortByAgeNonStatic(s1, s2));
            students3.forEach(in-> System.out.print(in.getName()+in.getAge()+"|"));
            System.out.println("第二种情况：传入实例方法");
    
    
            //第三种请款，不适用大括号
            List<Student> students4=getStudentList();
            Collections.sort(students4, (s1, s2) -> s1.getAge().compareTo(s2.getAge()));
            students4.forEach(in-> System.out.print(in.getName()+in.getAge()+"|"));
            System.out.println("第三种请款，不适用大括号");
    
    
            //第四种情况，使用大括号
            List<Student> students5=getStudentList();
            Collections.sort(students5, (s1, s2) -> {
                return s1.getAge().compareTo(s2.getAge());
            });
            students5.forEach(in-> System.out.print(in.getName()+in.getAge()+"|"));
            System.out.println("第四种情况，使用大括号");
    
        }
    
        private static List<Student> getStudentList() {
            List<Student> studentList = new ArrayList();
            for (int i = 0; i < 5; i++) {
                Student stu = new Student();
                stu.setAge(new Random().nextInt(10));
                stu.setName("tom");
                studentList.add(stu);
            }
            return studentList;
        }
    
    
    }
    
    
    /**
     * @author xiantao.wu
     * @create 2018/8/1515:44
     **/
    public class Student {
        private Integer age;
        private String name;
    
    
        public static int sortByAgeStatic(Student o1, Student o2) {
            return o1.getAge().compareTo(o2.getAge());
        }
    
    
        public int sortByAgeNonStatic(Student o1, Student o2) {
            return o1.getAge().compareTo(o2.getAge());
        }
    
    
        public Integer getAge() {
            return age;
        }
    
        public void setAge(Integer age) {
            this.age = age;
        }
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    }
    
### 测试结果： ####
    tom1|tom2|tom3|tom3|tom9|原生方法
    tom0|tom0|tom1|tom2|tom4|第一种情况：传入静态方法
    tom2|tom2|tom3|tom4|tom8|第二种情况：传入实例方法
    tom1|tom1|tom1|tom4|tom8|第三种请款，不适用大括号
    tom0|tom1|tom1|tom3|tom7|第四种情况，使用大括号