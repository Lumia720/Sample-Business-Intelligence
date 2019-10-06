# -*- coding: utf-8 -*-
"""
Created on Thu Jun 13 17:43:58 2019

@author: Yulia Pavlova
"""
import csv, collections #import library
import numpy as np
import datetime
#import timestring
#import dateutil


def read_csv(file_name):  #creating function to open file for reading and creating 2D array
    array_2D = []
    with open(file_name, 'r') as csvfile: 
        read = csv.reader(csvfile, delimiter=',') 
        for row in read:
            array_2D.append(row)
    return array_2D


data = read_csv('EVENT_TABLE.csv') #create a table and write its dimensions
data =np.array(data)
#[nrows,ncols] = np.shape(data)

nin = len(np.unique(data[:,0])) # number of unique number of invoices
counter=collections.Counter(data[:,0]) # keys and values of these invoices
invoices = list(counter.keys())
number_stages = list(counter.values())


# calculating the time differnce between invoice initiation and finilizing
invoice_flow = []
invoice_flow.append(invoices)
invoice_flow.append(number_stages)
timelength = []

for ind in invoices:
    index_in = np.where(data[:,0] == ind)
    t, p = np.shape(np.array(index_in))
    [de, leng] = np.shape(index_in)
    #extracting the initial and the closing times, fixin the date format inconsistency
    eventd =  data[index_in[0][0], 2]
    eventd = eventd.replace('/', '')
    try:
        t1=datetime.datetime.strptime(eventd, '%Y%m%d %H:%M:%S')
    except:
         try: 
             t1=datetime.datetime.strptime(eventd, '%d %b %Y %H:%M:%S')
         except: 
             try:
                 t1=datetime.datetime.strptime(eventd, '%d.%m.%Y %H:%M:%S')
             except:
                 t1=datetime.datetime.strptime(eventd, '%Y.%m.%d %H:%M:%S')
    eventd =  data[index_in[0][leng-1], 2]
    eventd = eventd.replace('/', '')
    try:
        t2=datetime.datetime.strptime(eventd, '%Y%m%d %H:%M:%S')
    except:
        try: t2=datetime.datetime.strptime(eventd, '%d %b %Y %H:%M:%S')
        except: 
             try:
                 t1=datetime.datetime.strptime(eventd, '%d.%m.%Y %H:%M:%S')
             except:
                 t1=datetime.datetime.strptime(eventd, '%Y.%m.%d %H:%M:%S')
    if t2-t1 < datetime.timedelta(0):
        print('Negtaive invoice proc timefor  invoice #',ind)
    td = t2-t1
    timelength.append(int(td.total_seconds()))

#sorting over the length of processing each invoice and writing into a new array
timelength=np.array(timelength)
invoice_flow.append(timelength)
invoice_flow = np.array(invoice_flow)
# the array below contains invoices, number of stages and timelength
invoice_flow_sort = invoice_flow[:,timelength.argsort()]

# open case table again
data2 = read_csv('CASE_TABLE.csv') #create a table and write its dimensions
data2 =np.array(data2)
vendor_vec = []
channel_vec = []
# extracting vendors and channels which correspond to invoices
for ind in invoice_flow_sort[0, :]:
    index_in = np.where(data2[:, 0] == ind)
    vendor_vec.append(data2[index_in[0][0],3])
    channel_vec.append(data2[index_in[0][0],7])

# appanding vendors and channels to invoice_flow_sort 
lst = list(invoice_flow_sort)
lst.append(vendor_vec)
lst.append(channel_vec)
invoice_flow_sort = np.asarray(lst)

#creating output file txt
with open('output.txt', 'w') as f:
    for item in invoice_flow_sort:
        f.write("%s\n" % item)
  
# creating new arrays vendors with average number of stages and average length of the invoice
vendors_av = []
channels_av = []
ven_st = []
ven_len = []
chan_st = []
chan_len = []

counter=collections.Counter(invoice_flow_sort[3, :]) # keys and values of these invoices
vendors = list(counter.keys())

for ven in vendors:
    index_ven = np.where(invoice_flow_sort[3, :] == ven)
    ven_st.append(np.average(np.array(invoice_flow_sort[1, index_ven]).astype(np.float)))
    ven_len.append(np.average(np.array(invoice_flow_sort[2, index_ven]).astype(np.float)))

vendors_av.append(vendors)
vendors_av.append(ven_st)
vendors_av.append(ven_len)    
    
counter=collections.Counter(invoice_flow_sort[4, :]) # keys and values of these invoices
channels = list(counter.keys())
    
for ch in channels:
    index_ch = np.where(invoice_flow_sort[4, :] == ch)
    chan_st.append(np.average(np.array(invoice_flow_sort[1, index_ch]).astype(np.float)))
    chan_len.append(np.average(np.array(invoice_flow_sort[2, index_ch]).astype(np.float)))

channels_av.append(channels)
channels_av.append(chan_st)
channels_av.append(chan_len) 


with open('output_vendors.txt', 'w') as f:
    for item in vendors_av:
        f.write("%s\n" % item)
        

with open('output_channels.txt', 'w') as f:
    for item in channels_av:
        f.write("%s\n" % item)


  