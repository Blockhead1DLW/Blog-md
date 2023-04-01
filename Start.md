---
title: A quick one click startup tool based on wxpython
tags: Tool
top: false
categories: Tool
---



------

```python
# -*- coding: utf-8 -*-
import os

###########################################################################
## Python code generated with wxFormBuilder (version 3.10.1-0-g8feb16b3)
## http://www.wxformbuilder.org/
##
## PLEASE DO *NOT* EDIT THIS FILE!
###########################################################################

import wx
import wx.xrc

###########################################################################
## Class mainFrame
###########################################################################

class mainFrame ( wx.Frame ):
    def __init__( self, parent ):
        wx.Frame.__init__ ( self, parent, id = wx.ID_ANY, title = u"Start --1.0", pos = wx.DefaultPosition, size = wx.Size( 250,600 ), style = wx.DEFAULT_FRAME_STYLE|wx.TAB_TRAVERSAL )

        self.SetSizeHints( wx.DefaultSize, wx.DefaultSize )

        sbSizer1 = wx.StaticBoxSizer( wx.StaticBox( self, wx.ID_ANY, wx.EmptyString ), wx.VERTICAL )

        gSizer2 = wx.GridSizer( 0, 2, 0, 0 )

        sbSizer4 = wx.StaticBoxSizer( wx.StaticBox( sbSizer1.GetStaticBox(), wx.ID_ANY, wx.EmptyString ), wx.VERTICAL )

        self.m_checkBox10 = wx.CheckBox( sbSizer4.GetStaticBox(), wx.ID_ANY, name[0][0], wx.DefaultPosition, wx.DefaultSize, 0 )
        self.m_checkBox10.SetValue(True)
        sbSizer4.Add( self.m_checkBox10, 0, wx.ALL, 5 )

        self.m_button2 = wx.Button( sbSizer4.GetStaticBox(), wx.ID_ANY, name[0][0], wx.DefaultPosition, wx.Size( 120,-1 ), 0 )
        sbSizer4.Add( self.m_button2, 0, wx.ALL, 5 )


        gSizer2.Add( sbSizer4, 1, wx.EXPAND, 5 )

        sbSizer41 = wx.StaticBoxSizer( wx.StaticBox( sbSizer1.GetStaticBox(), wx.ID_ANY, wx.EmptyString ), wx.VERTICAL )

        self.m_checkBox11 = wx.CheckBox( sbSizer41.GetStaticBox(), wx.ID_ANY, name[1][0], wx.DefaultPosition, wx.DefaultSize, 0 )
        self.m_checkBox11.SetValue(True)
        sbSizer41.Add( self.m_checkBox11, 0, wx.ALL, 5 )

        self.m_button21 = wx.Button( sbSizer41.GetStaticBox(), wx.ID_ANY, name[1][0], wx.DefaultPosition, wx.Size( 120,-1 ), 0 )
        sbSizer41.Add( self.m_button21, 0, wx.ALL, 5 )


        gSizer2.Add( sbSizer41, 1, wx.EXPAND, 5 )

        sbSizer42 = wx.StaticBoxSizer( wx.StaticBox( sbSizer1.GetStaticBox(), wx.ID_ANY, wx.EmptyString ), wx.VERTICAL )

        self.m_checkBox12 = wx.CheckBox( sbSizer42.GetStaticBox(), wx.ID_ANY, name[2][0], wx.DefaultPosition, wx.DefaultSize, 0 )
        self.m_checkBox12.SetValue(False)
        sbSizer42.Add( self.m_checkBox12, 0, wx.ALL, 5 )

        self.m_button22 = wx.Button( sbSizer42.GetStaticBox(), wx.ID_ANY, name[2][0], wx.DefaultPosition, wx.Size( 120,-1 ), 0 )
        sbSizer42.Add( self.m_button22, 0, wx.ALL, 5 )


        gSizer2.Add( sbSizer42, 1, wx.EXPAND, 5 )

        sbSizer43 = wx.StaticBoxSizer( wx.StaticBox( sbSizer1.GetStaticBox(), wx.ID_ANY, wx.EmptyString ), wx.VERTICAL )

        self.m_checkBox13 = wx.CheckBox( sbSizer43.GetStaticBox(), wx.ID_ANY, name[3][0], wx.DefaultPosition, wx.DefaultSize, 0 )
        self.m_checkBox13.SetValue(False)
        sbSizer43.Add( self.m_checkBox13, 0, wx.ALL, 5 )

        self.m_button23 = wx.Button( sbSizer43.GetStaticBox(), wx.ID_ANY, name[3][0], wx.DefaultPosition, wx.Size( 120,-1 ), 0 )
        sbSizer43.Add( self.m_button23, 0, wx.ALL, 5 )


        gSizer2.Add( sbSizer43, 1, wx.EXPAND, 5 )

        sbSizer44 = wx.StaticBoxSizer( wx.StaticBox( sbSizer1.GetStaticBox(), wx.ID_ANY, wx.EmptyString ), wx.VERTICAL )

        self.m_checkBox14 = wx.CheckBox( sbSizer44.GetStaticBox(), wx.ID_ANY, name[4][0], wx.DefaultPosition, wx.DefaultSize, 0 )
        self.m_checkBox14.SetValue(True)
        sbSizer44.Add( self.m_checkBox14, 0, wx.ALL, 5 )

        self.m_button24 = wx.Button( sbSizer44.GetStaticBox(), wx.ID_ANY, name[4][0], wx.DefaultPosition, wx.Size( 120,-1 ), 0 )
        sbSizer44.Add( self.m_button24, 0, wx.ALL, 5 )


        gSizer2.Add( sbSizer44, 1, wx.EXPAND, 5 )

        sbSizer45 = wx.StaticBoxSizer( wx.StaticBox( sbSizer1.GetStaticBox(), wx.ID_ANY, wx.EmptyString ), wx.VERTICAL )

        self.m_checkBox15 = wx.CheckBox( sbSizer45.GetStaticBox(), wx.ID_ANY, name[5][0], wx.DefaultPosition, wx.DefaultSize, 0 )
        self.m_checkBox15.SetValue(True)
        sbSizer45.Add( self.m_checkBox15, 0, wx.ALL, 5 )

        self.m_button25 = wx.Button( sbSizer45.GetStaticBox(), wx.ID_ANY, name[5][0], wx.DefaultPosition, wx.Size( 120,-1 ), 0 )
        sbSizer45.Add( self.m_button25, 0, wx.ALL, 5 )


        gSizer2.Add( sbSizer45, 1, wx.EXPAND, 5 )

        sbSizer46 = wx.StaticBoxSizer( wx.StaticBox( sbSizer1.GetStaticBox(), wx.ID_ANY, wx.EmptyString ), wx.VERTICAL )

        self.m_checkBox16 = wx.CheckBox( sbSizer46.GetStaticBox(), wx.ID_ANY, name[6][0], wx.DefaultPosition, wx.DefaultSize, 0 )
        self.m_checkBox16.SetValue(False)
        sbSizer46.Add( self.m_checkBox16, 0, wx.ALL, 5 )

        self.m_button26 = wx.Button( sbSizer46.GetStaticBox(), wx.ID_ANY, name[6][0], wx.DefaultPosition, wx.Size( 120,-1 ), 0 )
        sbSizer46.Add( self.m_button26, 0, wx.ALL, 5 )


        gSizer2.Add( sbSizer46, 1, wx.EXPAND, 5 )

        sbSizer47 = wx.StaticBoxSizer( wx.StaticBox( sbSizer1.GetStaticBox(), wx.ID_ANY, wx.EmptyString ), wx.VERTICAL )

        self.m_checkBox17 = wx.CheckBox( sbSizer47.GetStaticBox(), wx.ID_ANY, name[7][0], wx.DefaultPosition, wx.DefaultSize, 0 )
        self.m_checkBox17.SetValue(True)
        sbSizer47.Add( self.m_checkBox17, 0, wx.ALL, 5 )

        self.m_button27 = wx.Button( sbSizer47.GetStaticBox(), wx.ID_ANY, name[7][0], wx.DefaultPosition, wx.Size( 120,-1 ), 0 )
        sbSizer47.Add( self.m_button27, 0, wx.ALL, 5 )


        gSizer2.Add( sbSizer47, 1, wx.EXPAND, 5 )

        sbSizer48 = wx.StaticBoxSizer( wx.StaticBox( sbSizer1.GetStaticBox(), wx.ID_ANY, wx.EmptyString ), wx.VERTICAL )

        self.m_checkBox18 = wx.CheckBox( sbSizer48.GetStaticBox(), wx.ID_ANY, name[8][0], wx.DefaultPosition, wx.DefaultSize, 0 )
        self.m_checkBox18.SetValue(True)
        sbSizer48.Add( self.m_checkBox18, 0, wx.ALL, 5 )

        self.m_button28 = wx.Button( sbSizer48.GetStaticBox(), wx.ID_ANY, name[8][0], wx.DefaultPosition, wx.Size( 120,-1 ), 0 )
        sbSizer48.Add( self.m_button28, 0, wx.ALL, 5 )


        gSizer2.Add( sbSizer48, 1, wx.EXPAND, 5 )


        sbSizer49 = wx.StaticBoxSizer( wx.StaticBox( sbSizer1.GetStaticBox(), wx.ID_ANY, wx.EmptyString ), wx.VERTICAL )

        self.m_checkBox19 = wx.CheckBox( sbSizer49.GetStaticBox(), wx.ID_ANY, name[9][0], wx.DefaultPosition, wx.DefaultSize, 0 )
        self.m_checkBox19.SetValue(False)
        sbSizer49.Add( self.m_checkBox19, 0, wx.ALL, 5 )

        self.m_button29 = wx.Button( sbSizer49.GetStaticBox(), wx.ID_ANY, name[9][0], wx.DefaultPosition, wx.Size( 120,-1 ), 0 )
        sbSizer49.Add( self.m_button29, 0, wx.ALL, 5 )


        gSizer2.Add( sbSizer49, 1, wx.EXPAND, 5 )




        sbSizer410 = wx.StaticBoxSizer(wx.StaticBox(sbSizer1.GetStaticBox(), wx.ID_ANY, wx.EmptyString), wx.VERTICAL)

        self.m_checkBox110 = wx.CheckBox(sbSizer410.GetStaticBox(), wx.ID_ANY, name[10][0], wx.DefaultPosition, wx.DefaultSize, 0)
        self.m_checkBox110.SetValue(False)
        sbSizer410.Add(self.m_checkBox110, 0, wx.ALL, 5)

        self.m_button210 = wx.Button(sbSizer410.GetStaticBox(), wx.ID_ANY, name[10][0], wx.DefaultPosition,wx.Size(120, -1), 0)
        sbSizer410.Add(self.m_button210, 0, wx.ALL, 5)

        gSizer2.Add(sbSizer410, 1, wx.EXPAND, 5)

        sbSizer411 = wx.StaticBoxSizer(wx.StaticBox(sbSizer1.GetStaticBox(), wx.ID_ANY, wx.EmptyString), wx.VERTICAL)

        self.m_checkBox111 = wx.CheckBox(sbSizer411.GetStaticBox(), wx.ID_ANY, name[11][0], wx.DefaultPosition, wx.DefaultSize, 0)
        self.m_checkBox111.SetValue(True)
        sbSizer411.Add(self.m_checkBox111, 0, wx.ALL, 5)

        self.m_button211 = wx.Button(sbSizer411.GetStaticBox(), wx.ID_ANY, name[11][0], wx.DefaultPosition,
                                     wx.Size(120, -1), 0)
        sbSizer411.Add(self.m_button211, 0, wx.ALL, 5)

        gSizer2.Add(sbSizer411, 1, wx.EXPAND, 5)

        sbSizer1.Add( gSizer2, 1, wx.EXPAND, 5 )

        sbSizer3 = wx.StaticBoxSizer( wx.StaticBox( sbSizer1.GetStaticBox(), wx.ID_ANY, wx.EmptyString ), wx.VERTICAL )

        self.Start = wx.Button( sbSizer3.GetStaticBox(), wx.ID_ANY, u"Start", wx.DefaultPosition, wx.Size( 240,-1 ), 0 )
        sbSizer3.Add( self.Start, 0, wx.ALL, 5 )
        #
        # self.m_button20 = wx.Button( sbSizer3.GetStaticBox(), wx.ID_ANY, u"MyButton", wx.DefaultPosition, wx.Size( 240,-1 ), 0 )
        # sbSizer3.Add( self.m_button20, 0, wx.ALL, 5 )
        #
        # self.m_button211 = wx.Button( sbSizer3.GetStaticBox(), wx.ID_ANY, u"MyButton", wx.DefaultPosition, wx.Size( 240,-1 ), 0 )
        # sbSizer3.Add( self.m_button211, 0, wx.ALL, 5 )
        #
        # self.m_button221 = wx.Button( sbSizer3.GetStaticBox(), wx.ID_ANY, u"MyButton", wx.DefaultPosition, wx.Size( 240,-1 ), 0 )
        # sbSizer3.Add( self.m_button221, 0, wx.ALL, 5 )


        sbSizer1.Add( sbSizer3, 1, wx.EXPAND, 5 )


        self.SetSizer( sbSizer1 )
        self.Layout()

        self.Centre( wx.BOTH )

        # Connect Events
        self.m_button2.Bind( wx.EVT_BUTTON, self.bu1 )
        self.m_button21.Bind( wx.EVT_BUTTON, self.bu2 )
        self.m_button22.Bind( wx.EVT_BUTTON, self.bu3 )
        self.m_button23.Bind( wx.EVT_BUTTON, self.bu4 )
        self.m_button24.Bind( wx.EVT_BUTTON, self.bu5 )
        self.m_button25.Bind( wx.EVT_BUTTON, self.bu6 )
        self.m_button26.Bind( wx.EVT_BUTTON, self.bu7 )
        self.m_button27.Bind( wx.EVT_BUTTON, self.bu8 )
        self.m_button28.Bind( wx.EVT_BUTTON, self.bu9 )
        self.m_button29.Bind( wx.EVT_BUTTON, self.bu10 )
        self.m_button210.Bind( wx.EVT_BUTTON, self.bu11 )
        self.m_button211.Bind( wx.EVT_BUTTON, self.bu12 )
        self.Start.Bind( wx.EVT_BUTTON, self.start )

    def __del__( self ):
        pass

    # Virtual event handlers, override them in your derived class
    def bu1( self, event ):
        os.system("start %s"%name[0][1])
    def bu2( self, event ):
        os.system("start %s"%name[1][1])
    def bu3( self, event ):
        os.system("start %s"%name[2][1])
    def bu4( self, event ):
        os.system("start %s"%name[3][1])
    def bu5( self, event ):
        os.system("start %s"%name[4][1])
    def bu6( self, event ):
        os.system("start %s" % name[5][1])
    def bu7( self, event ):
        os.system("start %s" % name[6][1])
    def bu8( self, event ):
        os.system("start %s"%name[7][1])
    def bu9( self, event ):
        os.system("start %s" % name[8][1])
    def bu10( self, event ):
        os.system("start %s"%name[9][1])
    def bu11( self, event ):
        os.system("start %s"%name[10][1])
    def bu12( self, event ):
        os.system("start %s"%name[11][1])

    # start selected application
    def start( self, event ):
        if self.m_checkBox10.GetValue() == True:
            os.system("start %s"%name[0][1])
        if self.m_checkBox11.GetValue() == True:
            os.system("start %s"%name[1][1])
        if self.m_checkBox12.GetValue() == True:
            os.system("start %s"%name[2][1])
        if self.m_checkBox13.GetValue() == True:
            os.system("start %s"%name[3][1])
        if self.m_checkBox14.GetValue() == True:
            os.system("start %s"%name[4][1])
        if self.m_checkBox15.GetValue() == True:
            os.system("start %s"%name[5][1])
        if self.m_checkBox16.GetValue() == True:
            os.system("start %s"%name[6][1])
        if self.m_checkBox17.GetValue() == True:
            os.system("start %s"%name[7][1])
        if self.m_checkBox18.GetValue() == True:
            os.system("start %s"%name[8][1])
        if self.m_checkBox19.GetValue() == True:
            os.system("start %s"%name[9][1])
        if self.m_checkBox110.GetValue() == True:
            os.system("start %s"%name[10][1])
        if self.m_checkBox111.GetValue() == True:
            os.system("start %s"%name[11][1])
if __name__ == '__main__':
    name = []
    for line in open("start.txt", "r+"):
        tmp = line.split(',')
        name.append(tmp)
    app = wx.App(False)
    frame = mainFrame(None)
    frame.Show(True)
    app.MainLoop()
```