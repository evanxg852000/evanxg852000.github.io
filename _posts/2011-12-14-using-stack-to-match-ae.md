---
layout: post
title: "Using statck to match arithmetic expressions parentheses"
date: 2011-12-14 13:47:00
categories: [tutorial]
comments: true
---

This is an easy way of testing if an expression is correctly grouped using stack based method. 

<!--more-->

{% highlight cpp %}
#include <iostream>
#include <vector>
#include <string>
using namespace std;

int main (int argc, char *argv[]){
	vector<char> expr_validator;
	string expression="2[(a-9)/(2-b)]/(x-2)-[(4*(x-8)+4)/(5x+17)]";
	for(int i=0;i<expression.length() ;i++){
		switch(expression[i]){
			case '(':
					expr_validator.push_back(expression[i]);
				break;		
			case ')':
					if('('==expr_validator.back())
						expr_validator.pop_back();
					else
						expr_validator.push_back(expression[i]);
				break;
			case '[':
					expr_validator.push_back(expression[i]);
				break;
			case ']':
					if('['==expr_validator.back())
						expr_validator.pop_back();
					else
					expr_validator.push_back(expression[i]);
				break;					
		}
	}	
	
	if(expr_validator.empty()){
		cout << "Good Expression" << endl;	
	}else {
		cout << "Bad Expression" << endl;
		cout<<	expr_validator.size() ;
	}
	return 0;	
}
{% endhighlight %}


