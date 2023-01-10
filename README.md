# Crypto-Invester-Propine-Assessment
# Propine Assessment Problem:
# Objective:
For the given csv(comma seperated values) that represents a user daily transactions. it contains following information:(timestamps,transaction_type,token,amount).The program must be able to return 4 different type of result that is obtained from manipulation of the given csv data.The different set of results required are:

1.Given no parameters, return the latest portfolio value per token in USD
2.Given a token, return the latest portfolio value for that token in USD
3.Given a date, return the portfolio value per token in USD on that date
4.Given a date and a token, return the portfolio value of that token in USD on that date

# Data Description:
The data to be manipulated has :timestamps that are the Epoch date,transaction_type can be either DEPOSIT or WITHDRAW,token are only of three type i.e 'BTC','ETH','XRP',token amount

# Technologies:
framework library,npm,API used to obtain the intended result.
Node.js is an open-source, cross-platform, JavaScript runtime environment.
To read of data npm called 'fs' and to parse the csv file in manipulatable data "csv-parser" npm was used. as the data was fairly large, simple read of data was not possible the data was read as a stream line by line so it the program can operate at its own pace without memory allocation failure.
To ask quetions and recieve answers in real time i used npm called "readline" and to handle api call npm called"request".
As suggested by Mentors from Propine, "Cryptocompare" API was used to call for necessary real time data of token of various types and various date.

# Working process:
# For Condition 1:
After initial reading of stream data and asking questions were solved. i took first initiation to get the total sum of all the token information from the data given.A simple if statement sorts data according to its token and will either add to the token respective variable if its a type of "Deposit" else if its transaction type of "Withdraw" it will subtract from the said variable of the token.This operation usually took my okay computer about 90 seconds to go through all the 30 million data available:
    const checker=function(data){
        if(data.token==='BTC'&&data.transaction_type==='DEPOSIT'){
            BTCsum+=+data.amount;//BTCsum for BTC coin total
        }
        else if(data.token==='BTC'&&data.transaction_type==='WITHDRAWAL'){
            BTCsum-=+data.amount;
        }
        else if(data.token==='XRP'&&data.transaction_type==='DEPOSIT'){
            Xsum+=+data.amount;//Xsum for XRP coin total
        }
        else if(data.token==='XRP'&&data.transaction_type==='WITHDRAWAL'){
            Xsum-=+data.amount;
        }
        else if(data.token==='ETH'&&data.transaction_type==='DEPOSIT'){
            Esum+=+data.amount;//Esum for Ethereum coin total
        }
        else if(data.token==='ETH'&&data.transaction_type==='WITHDRAWAL'){
            Esum-=+data.amount;
        }
    }

This particular checking mechanism is quite often used so, i made it a function called "checker".This function is also used in condition 3
in line :

    BTCsum+=+data.amount;
+data.amount allowed conversion of string data to numeric one.

then the obtained values are multipled with todays token value obtained from cryptocompare.Then with careful word combination users could get the total value of all the coins in their possesion in real time.

# For condition 2:
readline will collect the user intended Token in variable called "token" and program will add data to variable called "sum" ,if the said token is of type "DEPOSIT" and will subtract from variable "sum" if data transaction type is of "WITHDRAW".At last after collection of total token in the whole portfolio Cryptocompare API real time data will be used to show the actual value of the said token in the current USD.
                if(data.token===token&&data.transaction_type==='DEPOSIT'){
                    sum+=+data.amount;
                    validDataUnderInputdate++;
                }
                else if(data.token===token&&data.transaction_type==='WITHDRAWAL'){
                    sum-=+data.amount;
                    validDataUnderInputdate++;
                }
# For condition 3:
This similar set of code is then used for specific date. The date in csv data are of Epoch but the users might not always know about the Epoch date that and would prefer their daily use date .So user could enter the date in GMT as such (06 Jan 2021 15:16:08 GMT) this date is then parsed and collected in a variable called ParsedDate.if data's epoch time is equal or smaller than the Entered date the total coin calucalted.This again cryptocompare api will be used with their historical data which also uses Epoch date for data retrieval.
                if(parseDate/1000 >= data.timestamp){
                    validDataUnderInputdate++;
                   checker(data);
                }
There was no method to call for multiple token historical data for a particular date at once.so multpile api call for each token was made.Sometime one api call may be faster than another and will affect the last "Totalvalue" of all the token in the users account.To prevent this ,i nested next api call in previous api call.So next api call will only be made once previous api call was successfully made.This allowed that the variable called "TotalValue" be correct.

                request({url:url},(error,res,body)=>{
                    if(!error&&res.statusCode==200){
                        const parseData=JSON.parse(body);
                        BTCvalue=parseData['Data']['Data'][0]['close']*BTCsum;
                        console.log(parseData['Data']['Data'][0]['close']);
                        console.log(`Total value of BTC $${BTCvalue.toLocaleString()}`);
                       
                        request({url:url2},(error,res,body2)=>{
                            if(!error&&res.statusCode==200){
                                const parseData2=JSON.parse(body2);
                                ETHvalue=parseData2['Data']['Data'][0]['close']*Esum;
                                console.log(parseData2['Data']['Data'][0]['close']);
                                console.log(`Total value of ETH $${ETHvalue.toLocaleString()}`);    

                                request({url:url3},(error,res,body)=>{
                                    if(!error&&res.statusCode==200){
                                        const parseData=JSON.parse(body);
                                        XRPvalue=parseData['Data']['Data'][0]['close']*Xsum;
                                        console.log(parseData['Data']['Data'][0]['close']);
                                        console.log(`Total value of XRP $${XRPvalue.toLocaleString()}`);  
                                        
                                        const Totalvalue=XRPvalue+BTCvalue+ETHvalue;
                                        console.log(`Total value of all the token in date: ${date} is $:${Totalvalue.toLocaleString()}`)
                                        console.log(`Valid number of data under input date ${validDataUnderInputdate}`)
                                    }
                                    })   
                            }
                            })
                    }
                    })
                
                     
            })
# For condition 4:
readline will ask for Date and the user intended token and store them in the variable called "ParseDate" and "Token" respectively. then similar to previous procedure the total token of respective token are collected in their respective variable and then multiplied with the crptocompare historical date of all the token and shown to user.
                if(parseDate/1000 >= data.timestamp){
                        
                        validDataUnderInputdate++;
                        if(data.token===token&&data.transaction_type==='DEPOSIT'){
                            sum+=+data.amount;
                        }
                        else if(data.token===token&&data.transaction_type==='WITHDRAWAL'){
                            sum-=+data.amount;
                        }
                        
                    }
# Minor adjustment in design:
To collect all the different condition for a proper user experience ,i carefully placed them in switch case and then again utlizied recursive calling of function.The use of recursive calling will help in error mitigation while entering number of different condition by user.
To verify the number of the data collected against the Date inserted ,i utilized use variable called "validDataUnderInputdate" which add if the required condition is fulfilled for the different condition.
To follow indutry security standard i have separated the apikey to the cryptocompare in a separate variable.But for convenience of Mentor i have kept the secret apikey in the same file as the main js file.
# Conclusion:
In this opportunity given by the Propine technology ,i collected and gained various knowledge as per how to read and manipulate large file of data with small memory .How to ask question directly from the terminal and write a readme to a program that you have built for others to understand your perspective to the design.
To make it work in a local Env
You must have Node and npm installed in your local machine .Clone this repo and then perform,
cd git-repo


npm install
to run the program ,input in the terminal

node main
To enter date of appropriate format ,or convert epoch to a appropraite date Epoch date Remove the Day given in front of the output.

i.e 1609946168=>Wed, 06 Jan 2021 15:16:08 GMT, but appropriate input is 06 Jan 2021 15:16:08 GMT.
