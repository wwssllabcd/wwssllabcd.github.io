---
layout: post
title: "如何在 subversion 下設定不同 repository 的存取權限"
date: 2012-08-03 17:58
comments: true
tags: Subversion
---
<!--more-->
有好心人士教我的，主要是加上  
	AuthzSVNAccessFile D:\svn\svn_access_file

例如:  
	DAV svn
	SVNParentPath d:/svn/
	AuthType Basic
	AuthName "Subversion repository"
	AuthUserFile D:\svn\svn-auth-file
	AuthzSVNAccessFile D:\svn\svn_access_file
	Require valid-user


而 svn_access_file 的樣子如下  
	[groups]
	cf_hw = ct_lin, jimmy, square
	lba_hw = ct_lin, vicent, yungwei
	ecc_hw = ct_lin, Gary
	[repository:/]
	@cf_hw = rw

	[LBA:/]
	@lba_hw = rw
	