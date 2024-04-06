<template>

<h1 class="mx-4 my-4 text-4xl font-bold text-center shadow p-6 shadow-cyan-600" >
    The Gauss-Seidel Method To Solve Linear System Equation
</h1>
<nav class="flex flex-row m-1 p-2 w-full">
    <input ref="ref_text_input" value="" type="number" class="basis-3/4" placeholder="Number Of Equations"/>
    <PrimaryButton class="basis-1/4 mx-2 justify-center " @click="createTable"> Create My Table </PrimaryButton>
</nav>

<div v-if="create_table" class="flex-1 mx-auto mt-9 mb-2 min-w-5/6 h-3/6  p-4 border ">
    <div class="overflow-auto">
        <table class="min-w-full" ref="table1">
            <thead class="bg-gray-100 border-b">
            <tr >
                <th v-for="n in p_number_of_equations" :key="n" class="px-6 py-3 text-left pr-0">X<p class="text-[10px] inline-flex">{{ n }}</p></th>
                <th class="px-6 py-3 text-left pr-0">C</th>
            </tr>
            </thead>
            <tbody>
            <tr v-for="id_of_eq in p_number_of_equations" :key="id_of_eq">
                <td v-for="n in (p_number_of_equations+1)" :key="n" class="px-6">
                    <input @input="assign_arr($event,id_of_eq,n)" type="number" class="m-1 p-1 w-1/4 min-w-16" required>
                </td>
            </tr>
            </tbody>
        </table>
    </div>

    <div class="flex flex-row border border-gray-200 my-4 mx-2 px-3 py-4 justify-center">
        <div class="mx-3">
             <input type="checkbox" class="mx-2" v-model="zeros">Zeros
            <input type="text"  class="mx-2" v-model="text_initial_val" placeholder="Initial Values: 1,2,3,4" :required="!zeros" :disabled="zeros" :class="zeros ? 'blur-sm' : ''">
        </div>
        <input type="number"  placeholder="Eps" v-model="Eps" required>
        <PrimaryButton class="mx-3 bg-green-600 hover:bg-green-300 active:bg-green-400 focus:bg-green-400" @click="solvefn">
            Solve
        </PrimaryButton>
    </div>
    <div v-if="error" class="text-red-600  text-center"> {{ error }}</div>

</div>

<!--        <PrimaryButton @click="test">-->
<!--            TestMe-->
<!--        </PrimaryButton>-->


 <div v-if="res" class="w-auto container items-center text-center p-2 shadow m-2">
        <p class="text-xl">
            {{res}}
        </p>
</div>

    <div v-if="solve" class="flex-1 my-4 mx-auto border border-gray-200 overflow-auto">
        <table class="min-w-full " ref="table2">
            <thead class="bg-gray-100 border border-b">
                <tr class="text-left">
                    <th>K</th>
                </tr>
             </thead>

            <tbody>
            <tr v-for="id_of_eq in p_number_of_equations" :key="id_of_eq">
                <td>{{id_of_eq}}</td>
            </tr>
            </tbody>
        </table>
    </div>
<!--    <pre>-->
<!--        {{arr}}-->
<!--        {{oldX}}-->
<!--    </pre>-->


</template>

<script setup>
import PrimaryButton from "@/Components/PrimaryButton.vue";
import {computed, nextTick, onMounted, ref, watch} from "vue";
import TextInput from "@/Components/TextInput.vue";

///

const number_of_equations = ref('');
const p_number_of_equations = computed(()=> { return parseInt(number_of_equations.value)})
const create_table = ref(false);
const zeros = ref(false)
const error = ref('');
///

////

const res_col_n = ref(0);
const table1 = ref();
const table2 = ref();
const solve = ref(false)
////
const res = ref(); // Store X as string
const Eps = ref(); // E the error
const ref_text_input = ref();
const arr = ref({})
const text_initial_val = ref("");
// const arr = ref({
//     0:[10,-1,2,0,6],
//     1:[-1,11,-1,3,25],
//     2:[2,-1,10,-1,-11],
//     3:[0,3,-1,8,15],
// }) // Store the linear equations

const initial_val = ref([]); // Store Initial  Values Of Equation
const oldX = ref(); // Store Xk-1
oldX.value = deepCopy(initial_val.value) // Copy Value of Initial Value
const X = ref(); // Store Xk
X.value = deepCopy(initial_val.value) // Copy Value of Initial Value

// Creating Deep Copy Function
function deepCopy(myValue){
    return JSON.parse(JSON.stringify(myValue))
}

function removeAllChildren(){
    const mytabel = table2.value;
    mytabel.querySelectorAll("thead tr th").forEach((value,key)=>{
        if(key === 0){
            console.log();
        }else{
            value.remove();
        }
    })

    mytabel.querySelectorAll("tbody tr").forEach((value,key) => {
        value.querySelectorAll("td").forEach((v,k)=>{
            if(k === 0){
                console.log();
            }else{
                v.remove();
            }
        })

    })
}
function resset(){
    oldX.value = deepCopy(initial_val.value);
    X.value = deepCopy(initial_val.value);
    if(res_col_n.value > 0){
        removeAllChildren()
    }

    res_col_n.value = 0 ;
    res.value = '';
    error.value = ''
}
function createTable(){
    if(create_table.value){
        table1.value.querySelectorAll("tbody tr").forEach((tdValue) => {
            tdValue.querySelectorAll("td").forEach((inputValue)=>{
                inputValue.querySelector('input').value = '';
            })
        })
    }

    create_table.value = false;
    arr.value = {};
    resset();
    solve.value = false
    number_of_equations.value = ref_text_input.value.value;
    if(p_number_of_equations.value > 0 ){
        create_table.value = true;
    }

}
function assign_arr(event,id,n){
    let eq = id - 1;
    let co = n - 1;

    let f_value = parseFloat(event.target.value);
    if(typeof arr.value[eq] == 'undefined') {
        arr.value[eq]  = [];
    }
    arr.value[eq][co] = f_value
}

function convertAndCheckNumbers(inputString) {
    // 1. Split the input string into an array of strings
    const stringArray = inputString.trim().split(/\s*,\s*/);

    // 2. Check if each element is a valid number
    for (const element of stringArray) {
        if (isNaN(Number(element))) {
            // Element is not a valid number, throw an error or handle it differently
            throw new Error(`Invalid number: ${element}`);
        }
    }

    // 3. Convert each string element to a number
    const numberArray = stringArray.map(Number);

    // 4. Return the resulting array of numbers
    return numberArray;
}
function assign_initial_values(){
    if(zeros.value){
        initial_val.value = new Array(p_number_of_equations.value).fill(0);
    }else{
        try {
            initial_val.value = convertAndCheckNumbers(text_initial_val.value);
        } catch (er) {
            error.value = er.message ; // Handle invalid input errors
        }
    }
}
// The Function Responsible on Calculating Xk with Gauss-Seidel Method
function assign_X(){

    // Iterate Throw arr to get id of equation and it's Coefficients
    Object.entries(arr.value).forEach(([key,value]) => {
        key = parseInt(key); // Convert key of arr to integer
        let result = 0.0; // Store the Result
        let dev = 1.0; // Store Coefficient of Current Xk
        // iterate throw Coefficients
        value.forEach((sValue,sKey) => {
            if(key === sKey){ // Get The Coefficient Of Current Xk
                dev = sValue;
            }else{
                if(sKey === value.length-1) { // Get The Absolute Coefficient
                    result += sValue;
                }else{
                    result -= sValue * X.value[sKey] ;
                }
            }
        })

        oldX.value[key] = deepCopy(X.value[key]); // Store Old X
        X.value[key] = result/dev; // Assign The New Value Of X

    })
    res_col_n.value++

    res.value = X.value.toString(); // Assign result
}


// The Function Check if The ||(Xk - Xk-1)|| / ||Xk|| <= E ;
function checkCondition(Eps)  {
       let res = X.value.filter((value,key) => {
         return Math.abs(value - oldX.value[key]) / Math.abs(value) > Eps;
    });
    return res.length === 0 ;
}

function add_value_to_result_table(){
    const mytabel = table2.value;
    const newheader = document.createElement('th');
    newheader.textContent = res_col_n.value;

    mytabel.querySelector("thead tr").appendChild(newheader);



    mytabel.querySelectorAll("tbody tr").forEach((value,key) => {
        const dataCell = document.createElement('td');
        dataCell.textContent = X.value[key];
        value.appendChild(dataCell);
    })

}
      // Optional: Perform any additional actions here if needed

function solvefn() {
    error.value = '';
    solve.value = false;
    res.value = '';
    if(Object.keys(arr.value).length === 0){
        error.value = "data is empty"
    }else {
        for (const key in arr.value) {
            if (arr.value[key].length === p_number_of_equations.value + 1) {
                console.log()
            } else {
                error.value = "enter all data"
                return;
            }

        }
    if(!(Eps.value >= 0)){
        error.value = "Eps must be zero or above"
    }else if(text_initial_val.value === '' && zeros.value === false){
        error.value = "Initial Values Is Empty"
    }
    assign_initial_values();
    if(error.value === ''){
        solve.value = true;
        setTimeout(()=>{
            calc()
        },100)
    }
    }


}
function calc(){
    resset();
    assign_X();
    while(!checkCondition(Eps.value)){
        add_value_to_result_table();
        assign_X();

    }
}

function test(){
    // arr.value = {
    //     0:[2,-1,-3],
    //     1:[-1,1,4],
    // };
    arr.value = {
        0:[10,-1,2,0,6],
        1:[-1,11,-1,3,25],
        2:[2,-1,10,-1,-11],
        3:[0,3,-1,8,15],
    }
    oldX.value = deepCopy(initial_val.value);
    X.value = deepCopy(initial_val.value);
    number_of_equations.value = 4;
    assign_initial_values()
    solvefn()
}

</script>
<style scoped>

</style>
