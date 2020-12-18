```typescript
class Student {
    fullName: string;
    constructor(public firstName, public middleInitial, public lastName) {
        this.fullName = firstName + " " + middleInitial + " " + lastName;
    }
}

interface Person {
    firstName: string;
    lastName: string;
}

function greeter(person : Person) {
    return "Hello, " + person.firstName + " " + person.lastName;
}

let user = new Student("Jane", "M.", 123);	// 不报错，因为ts是在编译时检查类型，不是在js运行时检查，没有运行代码结果

let p = {firstName: 'www', lastName: 333} // 报错

document.body.innerHTML = greeter(p);
```

