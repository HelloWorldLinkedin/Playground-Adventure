# Playground-Adventure
Advance self studies in swift 4. Self developed mini adventure game using playgrounds. 

import Foundation
public enum Jobs{
    case any
    case mage, wizard, sorcer
    case warrior, paladin, berserker
    case rogue, assassin, hunter
}

//Classes
public class Roles {
    public var name: Jobs
    public var health: Int
    public var mana: Int
    public var roleSkills: [Skills] = [basicAttack]
    public var allowedArmor: [ArmorType]
    public var allowedWeapon: [WeaponType]
    
    init (name: Jobs, health: Int, mana: Int, roleSkills: [Skills], allowedArmor: [ArmorType], allowedWeapon: [WeaponType]){
        self.name = name
        self.health = health
        self.mana = mana
        //Only add skills that of the same job
        for skills in roleSkills {
            if skills.role == name{
                self.roleSkills.append(skills)
            }
        }
        self.allowedArmor = allowedArmor
        self.allowedWeapon = allowedWeapon
    }
    public func addSkill (newSkill: Skills){
        if skillAllowed(newSkill: newSkill) == true && skillExist(newSkill: newSkill) == false{
            self.roleSkills.append(newSkill)
        }
    }
    public func skillAllowed (newSkill: Skills) -> Bool{
        if newSkill.role == name {
            return true
        }
        return false
    }
    public func skillExist (newSkill:Skills) -> Bool{
        for skill in roleSkills {
            if skill === newSkill {
                return true
            }
        }
        return false
    }
}

public var mage = Roles(name: Jobs.mage, health: 200, mana: 100, roleSkills: [fireball, lightning], allowedArmor: [ArmorType.linen, ArmorType.weaved], allowedWeapon: [WeaponType.unarmed, WeaponType.wand, WeaponType.staff])
import Foundation

public enum WeaponType{
    case unarmed
    case staff
    case wand
    case dagger
    case sword
    case axe
    case spear
}
public enum ArmorType{
    case linen, weaved, leather, light, heavy
}
//Additional Idea, Speed, bonus stats
public typealias weapon = (name: String, damage: Int, type: WeaponType)
public var fist: weapon = (name: "Fists", damage: 1, type: WeaponType.unarmed)
public var woodenWand: weapon = (name: "Wooden Wand", damage: 2, type: WeaponType.wand)
public var embuedWand: weapon = (name: "Embued Wand", damage: 5, type: WeaponType.wand)
public var holyStaff: weapon = (name: "Poking Stick", damage: 8, type: WeaponType.staff)
public var mageStaff: weapon = (name: "Mage Staff", damage: 15, type: WeaponType.staff)
public var battleMageStaff: weapon = (name: "Battle Mage Staff", damage: 20, type: WeaponType.staff)

public var woodenDagger: weapon = (name: "Wooden Dagger", damage: 2, type: WeaponType.dagger)
public var woodenAxe: weapon = (name: "Wooden Axe", damage: 4, type: WeaponType.axe)

public class Armor{
    public var name: String
    public var bonusHP: Int?
    public var bonusMP: Int?
    public var armorValue: Int
    public var type: ArmorType
    init(name: String, bonusHP: Int, bonusMP: Int, armorValue: Int, type: ArmorType) {
        self.name = name
        self.bonusHP = bonusHP
        self.bonusMP = bonusMP
        self.armorValue = armorValue
        self.type = type
    }
    convenience init (name: String, armorValue: Int, type: ArmorType) {
        self.init(name: name, bonusHP: 0, bonusMP: 0, armorValue: armorValue, type: type)
    }
}
public var cloth = Armor(name: "Cloth", armorValue: 1, type: ArmorType.linen)
public var apprenticeRobes = Armor(name: "Apprentice Robes", bonusHP: 20, bonusMP: 10, armorValue: 2, type: ArmorType.linen)
public var mageRobes = Armor(name: "Mage Robes", bonusHP: 30, bonusMP: 50, armorValue: 3, type: ArmorType.linen)

public var leatherCloth = Armor(name: "Leather", armorValue: 2, type: ArmorType.leather)

public var chainMail = Armor(name: "Chain Mail", armorValue: 3, type: ArmorType.light)

public var ironArmor = Armor(name: "Iron Armor", armorValue: 4, type: ArmorType.heavy)
public var steelrmor = Armor(name: "Steel Armor", armorValue: 5, type: ArmorType.heavy)
import Foundation


public func critCalculation(damage: Double) -> Int{
    return Int(damage * 1.75)
}

public func damageCalculation(attack: Int, defence: Int) -> Int{
    var result = attack - defence
    if attack <= defence {
        result = 1
    }
    
    return result
}
public func playerDamage(skill: Skills, player: Player, monster: Monster) -> Int{
    let playerAttack = skill.damage + player.currentWeapon.damage
    let monsterDefence = monster.defence
    
    return damageCalculation(attack: playerAttack, defence: monsterDefence)
}

public func randomInCounter() -> Monster {
    return monsterArray.randomElement()!
}

public func combat (monster: Monster, player: Player, playerSkill: Skills) {
    var playerAction = playerSkill
    repeat { //COMBAT
        //Combat check
        if player.currentMP < playerSkill.mana {
            playerAction = basicAttack
        }
        
        //Damage Calculation
        let damageReceived = damageCalculation(attack: monster.damage, defence: player.currentArmor.armorValue)
        let damageDealt = playerDamage(skill: playerAction, player: player, monster: monster)
        //Damage Resolution
        player.currentHP -= damageReceived
        player.currentMP -= playerAction.mana
        monster.health -= damageDealt
        player.updateStatus()
        
        //Combat Log
        print("\(playerAction.name) dealing \(damageDealt) to \(monster.name), \(monster.name)  dealing \(damageReceived), HP = \(player.currentHP)")
        
        if player.playerStatus == Status.alive {
            //print("\(player.name)'s CurrentHP = \(player.currentHP)")
            //Add gold
            player.currentGold += monster.goldDrop
//            if monster.health <= 0{
//                print("\(player.name) Slain \(monster.name)")
//            }
        }
    } while monster.health > 0 && player.playerStatus == Status.alive
    
    //reset monster
    monster.health = monster.maxHealth
}
import Foundation

//monster
public class Monster {
    public var name: String
    public var level: Int
    public var health: Int
    public var maxHealth: Int
    public var damage: Int
    public var defence: Int
    public var goldDrop: Int
    init (name: String, level: Int, health: Int, damage: Int, defence: Int){
        self.name = name
        self.level = level
        self.health = health
        self.maxHealth = health
        self.damage = damage
        self.defence = defence
        self.goldDrop = level*2
    }
}

public var imp = Monster(name: "Imp", level: 1, health: 10, damage: 5, defence: 0)
public var demonDog = Monster(name: "Demon Dog", level: 2, health: 20, damage: 6, defence: 1)
public var goblin = Monster(name: "Goblin", level: 3, health: 20, damage: 8, defence: 0)
public var armorGoblin = Monster(name: "Armored Goblin", level: 3, health: 50, damage: 6, defence: 3)
public var babyDragon = Monster(name: "Baby Dragon", level: 4, health: 10, damage: 10, defence: 10)
public var monsterArray: [Monster] = [imp, demonDog, goblin, armorGoblin, babyDragon]
import Foundation
public enum Status{
    case alive, dead
}

//Player
public class Player {
    public var name: String
    public var playerClass: Roles
    public var currentHP: Int
    public var currentMP: Int
    public var maxHP: Int
    public var maxMP: Int
    public var currentWeapon: weapon = fist
    public var currentArmor: Armor = cloth
    public var playerStatus: Status = Status.alive
    public var currentGold: Int = 0
    public var currentPercent: Double {
        let percent = Double(currentHP)/Double(maxHP) * 100.00
        //print("current = \(currentHP), max = \(maxHP), percent = \(percent)")
        return percent

    }

    public init (name: String, playerClass: Roles){
        self.name = name
        self.playerClass = playerClass
        currentHP = playerClass.health
        currentMP = playerClass.mana
        maxHP = currentHP
        maxMP = currentMP
    }
    public func updateStatus() {
        if currentHP <= 0 {
            playerStatus = Status.dead
        }
    }
    public func updateArmor (newArmor: Armor){
        if playerClass.allowedArmor.index(of: newArmor.type) != nil {
            
            if let newBonusHP = newArmor.bonusHP {
                if let currentBonusHP = currentArmor.bonusHP {
                    maxHP -= currentBonusHP
                    if currentHP > maxHP {
                        currentHP = maxMP
                    }
                }
                currentHP += newBonusHP
                maxHP += newBonusHP
            }
            if let newBonusMP = newArmor.bonusMP {
                if let currentBonusMP = currentArmor.bonusMP {
                    maxMP -= currentBonusMP
                    if currentMP > maxMP {
                        currentMP = maxMP
                    }
                }
                currentMP += newBonusMP
                maxMP += newBonusMP
            }
            currentArmor = newArmor
        }else{
            print("\(name) cannot equip \(newArmor.name)")
        }
    }
    public func updateWeapon (newWeapon: weapon){
        if playerClass.allowedWeapon.index(of: newWeapon.type) != nil {
            currentWeapon = newWeapon
        }else{
            print("\(name) cannot equip \(newWeapon.name)")
        }
    }
}
import Foundation

//Skills
public class Skills {
    public var name: String
    public var mana: Int
    public var damage: Int
    public var role: Jobs
    
    init (name: String, mana: Int,  damage: Int, role: Jobs){
        self.name = name
        self.mana = mana
        self.damage = damage
        self.role = role
    }
}
public class stunSkill: Skills {
    var stunChance: Int
    var stunTime: Int
    init (name: String, mana: Int,  damage: Int, role: Jobs, stunChance: Int, stunTime: Int){
        self.stunChance = stunChance
        self.stunTime = stunTime
        super.init(name: name, mana: mana, damage: damage, role: role)
    }
}
public class healingSkill: Skills{
    var heal: Int
    init(name: String, mana: Int, damage: Int, role: Jobs, heal: Int) {
        self.heal = heal
        super.init(name: name, mana: mana, damage: damage, role: role)
    }
}

public var basicAttack = Skills(name: "Attack", mana: 0, damage: 1, role: Jobs.any)

public var fireball = Skills(name: "Fire Ball", mana: 5, damage: 20, role: Jobs.mage)
public var lightning = stunSkill(name: "Lightning", mana: 7, damage: 30, role: Jobs.mage, stunChance: 10, stunTime: 1)

//Spell that can be purchased
public var fireCannon = Skills(name: "Flame Cannon", mana: 10, damage: 50, role: Jobs.mage)
public var fireBlast = Skills(name: "Fire Blast", mana: 15, damage: 80, role: Jobs.mage)

public var strike = Skills(name: "Strike", mana: 0, damage: 20, role: Jobs.warrior)
public var heroicStrike = Skills(name: "Heroic Strike", mana: 10, damage: 50, role: Jobs.warrior)

public var consecration = healingSkill(name: "Consecration", mana: 10, damage: 20, role: Jobs.paladin, heal: 10)
public var grandCross = healingSkill(name: "Grand Cross", mana: 10, damage: 40, role: Jobs.paladin, heal: 15)
