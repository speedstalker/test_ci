#!/usr/bin/env groovy

def buildUnix(label, stash_label) {
	node(label) {
		checkout scm
		sh """#!/bin/sh
			gcc hello.c -o hello_${label}.out
		"""
		stash name: stash_label, includes: "*.out"
		deleteDir()
	}
}

timestamps {
	stage('build') {
		parallel(
			"build_win": {
				node("windows") {
					checkout scm
					String vsvars_bat = 'Microsoft Visual Studio 12.0\\VC\\vcvarsall.bat'
					bat """
						call "%ProgramFiles(X86)%\\${vsvars_bat}" x86
						cl.exe hello.c
					"""
					stash name: "build_win", includes: "*.exe"
					deleteDir()
				}
			},
			"build_mac": {
				buildUnix("mac", "build_mac")
			},
			"build_lin": {
				buildUnix("linux", "build_lin")
			}
		)
	}
	stage('store') {
		node("master") {
			unstash "build_mac"
			unstash "build_win"
			unstash "build_lin"
			archiveArtifacts artifacts: "*", onlyIfSuccessful: true
			deleteDir()
		}
	}
}
