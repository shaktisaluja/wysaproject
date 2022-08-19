let express=require("express")
let router=express.Router();
const mongoose=require("mongoose")

const sleepSchema =new mongoose.Schema({
    nickName:{
        type:String,
        required:"true",
        trim:true
    },
    changeAfterBetterSleep:{
      type:String,
      required:"true",
      enum:[
        "I would go to sleep eaisly",
        "I would sleep through the night",
        "I'd wake up on time,refreshed"
      ]
    },
    strugglingPeriod:{
      type:String,
      required:"true",
      enum:[
        "Less than 2 weeks",
        "2 to 8 weeks",
        "More than 8 weeks"
    ]
    },
    bedTime:{
        type:String,
        required:"true",
    },
    awakeTime:{
        type:String,
        required:"true",
    },
    totalSleepHour:{
        type:String,
        required:"true",
       

    }
}, { timestamps: true });



const dataModel=mongoose.model("Sleep",sleepSchema)

//---------------------validations--------------------------------------------------------

const isValid = function (value) {
    if (typeof value === "undefined" || typeof value === null) return false
    if (typeof value === "string" && value.trim().length == 0) return false
    return true
}



const isValidTime=function(value){
    let front=+value.slice(0,2),mid=+value.slice(3,5),back=value.slice(-2)
    if(value[2]!=":")return false
    if(front<1 || front>12)return false
    if(mid<0 || mid>59)return false
    if(back!="am" && back!="pm")return false

    return true
}

const isValidSleepHour=function(value){
    let time=+value.slice(0,2),hour=value.slice(-3)
    if(time>10 || time<0)return false
    if(hour!="hrs")return false
     return true
}


//--------------------------------------TimeDifference in minute calculation------------
let timeDifference=(start,end)=>{
    let t1=start.slice(-2),t2=end.slice(-2)

    //hours and minutes of start and end
    let front1=+start.slice(0,2),back1=+start.slice(3,5)
    let front2=+end.slice(0,2),back2=+end.slice(3,5)

    let res
    //Mathematical calculation of time difference of start and end in minutes
    if(t1=="pm" && t2=="pm"){
      res=(12-(front1+1))*60+(60-back1)+(12+front2)*60+back2
    }
    else if(t1=="am" && t2=="am"){
        res=front2*60+back2-(front1*60+back1)
      }
    else {
        res=(12-(front1+1))*60+(60-back1)+front2*60+back2
    }
    
    return res
    }
//--------------------------------------------------------------------------------------


router.post("/sleepEffenciency",async(req,res)=>{
try{
    let data =req.body
    let {nickName,strugglingPeriod,bedTime,awakeTime,totalSleepHour}=data
    
    
    if (Object.keys(data).length == 0) {
        return res.status(400).send({ status: false, message: "EMPTY INPUT" })
    }
    if (!isValid(nickName)) {
        return res.status(400).send({ status: false, message: "Please enter valid Nickname" })
    }
    if (!isValid(strugglingPeriod)) {
        return res.status(400).send({ status: false, message: "Please enter valid strugglingPeriod" })
    }
    if (!isValid(bedTime)) {
        return res.status(400).send({ status: false, message: "Please enter valid bedTime" })
    }
    if (!isValid(awakeTime)) {
        return res.status(400).send({ status: false, message: "Please enter valid awakeTime" })
    }
    if (!isValidTime(bedTime)) {
        return res.status(400).send({ status: false, message: "Please enter valid bedTime" })
    }
    if (!isValidTime(awakeTime)) {
        return res.status(400).send({ status: false, message: "Please enter valid awakeTime" })
    }
    if (!isValidSleepHour(totalSleepHour)) {
        return res.status(400).send({ status: false, message: "Please enter valid totalSleepHour" })
    }

    //Calculation of effenciency
    let timeInBed= timeDifference(bedTime,awakeTime)
    let sleepMinute= +totalSleepHour.slice(0,2)
    let effenciency=parseInt((sleepMinute*60*100)/timeInBed)

    if(effenciency>100)effenciency=100
    else effenciency=effenciency
    data["effenciency"]=effenciency

    let msg= effenciency!=100?"We'll get this up to 90% ":"Thats great"

      let saved = await dataModel.create(data)
    return res.status(201).send({message:`You seem to have sleep efficiency of ${effenciency}%  ${msg} 😎......A higher sleep efficiency score means a more refreshing and energizing sleep,which can help you move into your day with a sense of lightness and ease`})
   
 
  

}catch(err){
    res.status(500).send({ status: false, message: err.message })
}
})


module.exports={isValid,isValidSleepHour,isValidTime}