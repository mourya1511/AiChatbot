from flask import Flask,render_template,request,redirect
from werkzeug.wrappers import response
flask_object = Flask(_name_)
import firebase_admin
from firebase_admin import credentials
from firebase_admin import db
from chatterbot import ChatBot
from chatterbot.trainers import ChatterBotCorpusTrainer
my_bot = ChatBot('PyBot',
    storage_adapter='chatterbot.storage.SQLStorageAdapter',
    logic_adapters=[
        {
            'import_path': 'chatterbot.logic.BestMatch',
            'default_response': 'I am sorry, but I do not understand.',
            'maximum_similarity_threshold': 1.50
        }
    ]
)

name = "";
#trainer = ChatterBotCorpusTrainer(my_bot)
#trainer.train('chatterbot.corpus.english')

charAryVar = [{}]

cred = credentials.Certificate("fbkey.json")
firebase_admin.initialize_app(cred,
#{'databaseURL':'https://aichatbot-6942c-default-rtdb.firebaseio.com/'})
{'databaseURL':'https://chatbot-ba02b-default-rtdb.firebaseio.com/'})
dbref=db.reference("/")

# [==] CodTdo
@flask_object.route('/', methods=['GET','POST'])
def operfnc():
    global name;
    global charAryVar;
    if(request.form):
        tag = request.form['tag'].lower()
        if tag=="visitor":
            return render_template("contact_form.html")
        elif tag=="administrator":
            return render_template("Admin_form.html")  
        elif tag=="back":
            return redirect("/")
        else:
            if(tag != ""):
                resp = my_bot.get_response(tag)
                qryval = 0
                if(str(resp) == "I am sorry, but I do not understand."):
                    print("Its new Query")
                    dbref3=db.reference("/NewQueries")
                    NewQueriescnt=dbref3.get()
                    for index,col in enumerate(NewQueriescnt) :
                         if index > 0:
                             if col["query"] == tag:
                                 qryval = 1;
                                 break;
                    if(qryval == 0):
                        NewQueriescnt.append({'query':tag})
                        dbref3.set(NewQueriescnt)           
                charAryVar.append({"id":len(charAryVar)+1,
                                "user":"me",
                                "msg":tag},
                                )

                charAryVar.append({"id":len(charAryVar)+1,
                                "user":"bot",
                                "msg":resp},
                                )
                return render_template("chatbot.html",ip=charAryVar,un=name)    
    else:
        return render_template("college.html")

@flask_object.route('/adminlogin', methods=['POST'])
def loginfnc():
    if(request.form):
        username= request.form['username']
        password= str(request.form['password'])
        tag=request.form['login']
        if tag=="login":
            dbref=db.reference("")
            dbcnt=dbref.get()
            susername=dbcnt["username"]
            spassword=dbcnt["password"]
            print(susername,spassword)
            if (username==susername and password==spassword):
                resp="login successful!.."
                dbref2=db.reference("/Visitors")
                userscnt=dbref2.get()
                dbref3=db.reference("/NewQueries")
                NewQueriescnt=dbref3.get()
                return render_template("table.html",dbdata=userscnt,QryData=NewQueriescnt)    
            else:
                resp="The username/password incorrect.."
                return render_template("Admin_form.html",ip=resp)
        elif tag=="back":
            return redirect("/")
    else:
        return render_template("Admin_form.html")

@flask_object.route('/contact', methods=['POST'])
def contactfetch():
    global name;
    if(request.form):
        uev = 0;
        global charAryVar ;
        charAryVar = [{"id":0,
                       "user":"bot",
                        "msg":"""Hi, Greetings From ,\n How can I 
                        /n help you"""}];
        name= request.form['nametag']
        phone= request.form['Phonenotag']
        email= request.form['Emailtag']
        gender= request.form['Gendertag']
        branch= request.form['Branchtag']
        dbref2=db.reference("/Visitors")
        dbcnt=dbref2.get()

        for index,col in enumerate(dbcnt) :
            if index > 0:
                if col["phone"] == phone:
                    uev = 1;
                    break;
        if uev == 0:
            dbcnt.append({'name': name, 'phone':phone,'email':email ,'gender':gender, 'branch':branch})
            dbref2.set(dbcnt)
        return render_template("chatbot.html",ip=charAryVar,un=name)
    else:
        return render_template("college.html")

@flask_object.route('/logout', methods=['POST'])
def authoper():
    if(request.form):
        authoprval= request.form['btnval']
        if (authoprval=="DeleteUsers"):
            dbref=db.reference("/Visitors")
            dbref.delete()
            dbref.set([{"TestUser":123}])
            dbref2=db.reference("/Visitors")
            userscnt=dbref2.get()
            return render_template("table.html",actionresp="Deleted all Visitors Infromation",dbdata=userscnt)
        if (authoprval=="DeleteQueries"):
            dbref=db.reference("/NewQueries")
            dbref.delete()
            dbref.set([{"Test":123}])
            dbref3=db.reference("/NewQueries")
            NewQueriescnt=dbref3.get()
            return render_template("NewQueriestable.html",actionresp="Deleted all Queries",QryData=NewQueriescnt)
        if (authoprval=="NewQueries"):
            dbref3=db.reference("/NewQueries")
            NewQueriescnt=dbref3.get()
            return render_template("NewQueriestable.html",QryData=NewQueriescnt)
        else:
            return redirect("/")
    else:
        return render_template("table.html")    

if _name_ == '_main_':
    flask_object.run(debug=True,port = 5052)
