# Class
```ts
class User{

    public readonly name:string;
    private age:number;
    private passwd:string;
	protected id:number;


    constructor(
        public readonly name:string,
        age:number
    ){

        this.name=name;
        this.age=age;

    }

}
```
```ts
abstract class Animal{


abstract sound():void;


move(){

console.log("移动")

}

}
```

```ts
interface Flyable{

fly():void;

}
class Bird implements Flyable {
	fly(){
	}
}

```


```ts
class Box<T>{


value:T;


constructor(value:T){

this.value=value;

}

}


```
```ts
const a=new Box<number>(100);

const b=new Box<string>("hello");
```