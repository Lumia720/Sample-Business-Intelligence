# -*- coding: utf-8 -*-
"""
Created on Thu Jun 13 17:43:58 2019

@author: Yulia Pavlova
"""
import csv, collections #import library
import numpy as np
def read_csv(file_name):  #creating function to open file for reading and creating 2D array
    array_2D = []
    with open(file_name, 'r') as csvfile: 
        read = csv.reader(csvfile, delimiter=',') 
        for row in read:
            array_2D.append(row)
    return array_2D


data = read_csv('CASE_TABLE.csv') #create a table and write its dimensions
data =np.array(data)
[nrows,ncols] = np.shape(data)
nch = len(np.unique(data[:,ncols-1])) # number of unique number of channels
counter=collections.Counter(data[:,ncols-1]) # keys and values of these channele
channels = list(counter.keys())
channels_total_invoices = list(counter.values())


#calculating proc. flow: channels, invoices, spendings
proc_flow = []
proc_flow.append(channels)
proc_flow.append(channels_total_invoices)
spends = []
for ch in channels:
    index_ch = np.where(data[:,ncols-1] == ch)
    t, p = np.shape(np.array(index_ch))
    spends_ch = 0;
    for i in (0,p-1): spends_ch = spends_ch + float(data[index_ch[0][i],5])*float(data[index_ch[0][i],6])
    spends.append(spends_ch) 

proc_flow.append(spends)
print(proc_flow)
# add also users and vendors, who buys and sells most and least